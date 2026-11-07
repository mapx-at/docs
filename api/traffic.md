# Verkehr

Ereignismeldungen (Unfälle, Baustellen, Sperren), Verkehrsregelungen, Wechselverkehrszeichen und Routing mit Sperren aus offiziellen österreichischen Quellen.

Die Verkehrs-API liefert Ereignismeldungen (Unfälle, Baustellen, Sperren mit Zeitraum) und Verkehrsregelungen für ganz Österreich, den Stand der ASFINAG-Wechselverkehrszeichen auf Autobahnen und Schnellstraßen als GeoJSON oder Vektorkacheln und das Routing-Profil `driving-traffic`, das Sperren umfährt. Kein Schlüssel, CORS offen.

> Quellen: **EVIS.AT** der ASFINAG (Ereignisse geplant und ungeplant, Verkehrsregelungen von Ländern, Städten, Polizei und ASFINAG; CC BY 4.0 nach Registrierung) und die **Wechselverkehrszeichen** der ASFINAG (CC BY 4.0). Das Wegenetz stammt von [GIP.at](https://www.gip.at) (CC BY 4.0). Namensnennung: „Verkehrsdaten: ASFINAG / EVIS.AT“. Jede Antwort enthält das Feld `attribution`.
> 
> Meldungen mit `mandatory: true` (EVIS `compliance-option` mandatory) und alle Verkehrsregelungen sind verbindlich: der Router behandelt sie als Sperre, und Karten müssen sie unverändert darstellen (Text `text` nicht kürzen oder umdeuten). Nicht verwendet werden die kostenpflichtigen EVIS-Daten zur flächigen Verkehrslage, Prognose und Ganglinien; es gibt daher keine Staustufen.

EVIS wird zentral im Routing-Dienst abgerufen (Prüfsumme alle 2 Minuten, Daten nur bei Änderung), die Wechselverkehrszeichen alle 2 Minuten. Anfragen an die mapx-API belasten die Quellen nicht.

## Ereignismeldungen

**GET https://api.mapx.at/v1/traffic/incidents**

<ParamTable params={[
  { name: "bbox", type: "string", desc: "<code>west,süd,ost,nord</code> in Grad. Ohne <code>bbox</code> kommen alle Meldungen, auch die ohne Ortsangabe." },
  { name: "planned", type: "boolean", desc: "<code>true</code> nur geplante, <code>false</code> nur ungeplante Ereignisse." },
  { name: "road", type: "string", desc: "Straßennummer, zum Beispiel <code>A12</code>." },
  { name: "active", type: "boolean", default: "true", desc: "<code>false</code> liefert auch beendete und künftige Meldungen." },
]} />

Antwort: GeoJSON `FeatureCollection` mit der Geometrie entlang der GIP-Kanten (`LineString` oder `MultiLineString`).

<ParamTable kind="response" params={[
  { name: "kind, category", type: "string", desc: "Art, zum Beispiel <code>Straßensperre</code>, <code>Bauarbeiten</code>, <code>Unfall</code>, <code>Verkehrssteuerungsmaßnahme</code>." },
  { name: "mandatory", type: "boolean", desc: "Verbindliche Meldung: gilt im Routing als Sperre." },
  { name: "both_directions", type: "boolean", desc: "Wirkt in beiden Fahrtrichtungen." },
  { name: "text", type: "string", desc: "Meldungstext der ASFINAG." },
  { name: "road", type: "string", desc: "Straßennummer." },
  { name: "planned, closed", type: "boolean", desc: "Geplantes Ereignis, Straße vollständig gesperrt." },
  { name: "start, end, updated", type: "string", desc: "Zeitraum und letzte Änderung, UTC." },
]} />

<Code lang="bash" code={`# Meldungen auf der A12 (Inntal)
curl "https://api.mapx.at/v1/traffic/incidents?road=A12"`} />

## Tempolimits (Wechselverkehrszeichen)

**GET https://api.mapx.at/v1/traffic/flow**

<ParamTable params={[
  { name: "bbox", type: "string", required: true, desc: "<code>west,süd,ost,nord</code> in Grad." },
]} />

Autobahn- und Schnellstraßen-Kanten, auf denen ein Wechselverkehrszeichen mit Position ein Tempolimit unter der freien Geschwindigkeit anzeigt (`speed_kmh`, `free_kmh`, `level` 0 bis 3 nach Verhältnis). Das Limit gilt im Router auf der Kante am Schild, nicht auf der ganzen Strecke bis zum nächsten Zeichen.

## Vektorkacheln und Stil

**GET https://api.mapx.at/v1/traffic/tiles.json**

Die Kacheln `/v1/traffic/tiles/{z}/{x}/{y}.pbf` (Zoom 5 bis 14, 60 Sekunden Cache) enthalten die Ebenen `traffic` (`level`, `speed`, `free`) und `incidents` (`kind`, `text`, `road`, `planned`, `closed`). Die Stile `karte*` enthalten die Ebenen `verkehr-*` ausgeblendet (`visibility: none`); der Stil `karte-verkehr` zeigt sie. Zum Einblenden: `map.setLayoutProperty("verkehr-lage", "visibility", "visible")` für alle Ebenen mit Präfix `verkehr-`.

## Routing mit Sperren

Das Profil `driving-traffic` gilt für `directions`, `matrix` und `isochrone`. Verbindliche Sperren (EVIS `mandatory`, Verkehrsregelungen für Pkw) sind gesperrt. Unverbindliche Straßensperren und zeitlich bedingte Regelungen (Wochentage und Uhrzeiten stehen nur im Text) werden stark gemieden, bleiben aber befahrbar, damit das Netz zusammenhängt; die Fahrzeit bleibt unverändert. Tempolimits der Wechselverkehrszeichen verlängern die Fahrzeit. Es gibt keine flächige Verkehrslage: übrige Kanten behalten die Standardgewichte. Der Router rechnet ohne Contraction Hierarchies und ist langsamer als `driving`; ohne Datenstand antwortet er mit `503`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/driving-traffic/10.74,47.24;10.66,47.07"`} />

## Wechselverkehrszeichen

**GET https://api.mapx.at/v1/traffic/signs**

Aktueller Zustand der ASFINAG-Wechselverkehrszeichen: `speed_kmh` (angezeigtes Tempolimit), `pictogram`, `text` (Zusatztexte), `image` (Bild der ASFINAG), `working`. Die Quelle nennt eine Position nur bei einem kleinen Teil der Anlagen (rund 40 von 5300); die übrigen verweisen auf die Anlagentabelle der ASFINAG und erscheinen nur in `total_signs`. Optional `bbox`.

## Status

**GET https://api.mapx.at/v1/traffic/status**

Zeigt Stand der Daten (`fetched`), Zahl der Meldungen und Abschnitte sowie die im Router gesperrten und verlangsamten Kanten.
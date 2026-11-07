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
  { name: "active", type: "boolean", default: "true", desc: "<code>false</code> liefert auch beendete und künftige Meldungen (Standard, sobald <code>from</code> oder <code>to</code> gesetzt ist)." },
  { name: "type", type: "string", desc: "Kommagetrennt: <code>road_closure</code>, <code>construction</code>, <code>lane_restriction</code>, <code>accident</code>, <code>congestion</code>, <code>road_hazard</code>, <code>miscellaneous</code>." },
  { name: "from, to", type: "string", desc: "Zeitfenster in UTC, <code>YYYY-MM-DD</code> oder <code>YYYY-MM-DDTHH:MMZ</code>: Meldungen, deren Zeitraum das Fenster berührt." },
  { name: "route", type: "string", desc: "Nur Meldungen entlang einer Route: Encoded Polyline (wie <code>geometries=polyline6</code>) oder GeoJSON-<code>LineString</code>. Berücksichtigt die Fahrtrichtung; sortiert nach Lage auf der Route." },
  { name: "route_precision", type: "number", default: "6", desc: "Präzision der Polyline, 5 oder 6." },
  { name: "buffer", type: "number", default: "25", desc: "Korridorbreite je Seite in Meter (1 bis 1000)." },
  { name: "limit, offset", type: "number", default: "200, 0", desc: "Seitenweise Abfrage, <code>limit</code> höchstens 1000. Die Antwort nennt <code>total</code> und, falls weitere folgen, <code>next_offset</code>." },
]} />

Lange Routen passen nicht in eine URL: dieselben Parameter gehen auch als JSON-Objekt per `POST` (dort darf `route` ein GeoJSON-Objekt sein).

Antwort: GeoJSON `FeatureCollection` mit der Geometrie entlang der GIP-Kanten (`LineString` oder `MultiLineString`).

<ParamTable kind="response" params={[
  { name: "type, impact", type: "string", desc: "Typ im Mapbox-Wortschatz (siehe <code>type</code>) und Schwere: <code>critical</code> (gesperrt), <code>major</code> (verbindlich, Fahrstreifen, Unfall), <code>minor</code>, <code>low</code> (geplant). Von mapx aus Art und Sperrangabe abgeleitet." },
  { name: "geometry_index_start, geometry_index_end", type: "number", desc: "Nur mit <code>route</code>: betroffene Stützpunkte der übergebenen Route." },
  { name: "kind, category", type: "string", desc: "Art, zum Beispiel <code>Straßensperre</code>, <code>Bauarbeiten</code>, <code>Unfall</code>, <code>Verkehrssteuerungsmaßnahme</code>." },
  { name: "mandatory", type: "boolean", desc: "Verbindliche Meldung: gilt im Routing als Sperre." },
  { name: "both_directions", type: "boolean", desc: "Wirkt in beiden Fahrtrichtungen." },
  { name: "text", type: "string", desc: "Meldungstext der ASFINAG." },
  { name: "road", type: "string", desc: "Straßennummer." },
  { name: "planned, closed", type: "boolean", desc: "Geplantes Ereignis, Straße vollständig gesperrt." },
  { name: "start, end, updated", type: "string", desc: "Zeitraum und letzte Änderung, UTC." },
]} />

<Code lang="bash" code={`# Meldungen auf der A12 (Inntal)
curl "https://api.mapx.at/v1/traffic/incidents?road=A12"

# Sperren und Baustellen im Dezember rund um Linz
curl "https://api.mapx.at/v1/traffic/incidents?bbox=14.1,48.2,14.5,48.4&type=road_closure,construction&from=2026-12-01&to=2026-12-31"

# Meldungen entlang einer Route (Geometrie aus /v1/directions mit geometries=polyline6)
curl -X POST https://api.mapx.at/v1/traffic/incidents -H "Content-Type: application/json" \\
  -d '{"route": {"type": "LineString", "coordinates": [[16.37, 48.21], [16.30, 48.19]]}, "buffer": 30}'`} />

## Tempolimits (Wechselverkehrszeichen)

**GET https://api.mapx.at/v1/traffic/flow**

<ParamTable params={[
  { name: "bbox", type: "string", required: true, desc: "<code>west,süd,ost,nord</code> in Grad." },
]} />

Autobahn- und Schnellstraßen-Kanten, auf denen ein Wechselverkehrszeichen mit Position ein Tempolimit unter der freien Geschwindigkeit anzeigt (`speed_kmh`, `free_kmh`, `level` 0 bis 3 nach Verhältnis). Das Limit gilt im Router auf der Kante am Schild, nicht auf der ganzen Strecke bis zum nächsten Zeichen.

## Vektorkacheln und Stil

**GET https://api.mapx.at/v1/traffic/tiles.json**

Die Kacheln `/v1/traffic/tiles/{z}/{x}/{y}.pbf` (Zoom 5 bis 14, 60 Sekunden Cache) enthalten die Ebenen `traffic` (`level`, `speed`, `free`) und `incidents` (`kind`, `text`, `road`, `planned`, `closed`). Die Stile `streets*` enthalten die Ebenen `verkehr-*` ausgeblendet (`visibility: none`); der Stil `traffic-day` zeigt sie. Zum Einblenden: `map.setLayoutProperty("verkehr-lage", "visibility", "visible")` für alle Ebenen mit Präfix `verkehr-`. Im Viewer auf mapx.at schaltet „Ebenen → Verkehr“ die Ebenen ein (standardmäßig aus).

## Routing mit Sperren

Das Profil `driving-traffic` gilt für `directions`, `matrix` und `isochrone`. Verbindliche Sperren (EVIS `mandatory`, Verkehrsregelungen für Pkw) sind gesperrt. Unverbindliche Straßensperren und zeitlich bedingte Regelungen (Wochentage und Uhrzeiten stehen nur im Text) werden stark gemieden, bleiben aber befahrbar, damit das Netz zusammenhängt; die Fahrzeit bleibt unverändert. Tempolimits der Wechselverkehrszeichen verlängern die Fahrzeit. Es gibt keine flächige Verkehrslage: übrige Kanten behalten die Standardgewichte. Der Router rechnet ohne Contraction Hierarchies und ist langsamer als `driving`; ohne Datenstand antwortet er mit `503`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/driving-traffic/10.74,47.24;10.66,47.07"`} />

**Meldungen in der Route:** Bei `driving-traffic` (und mit `incidents=true` auch bei `driving`) enthält jede Teilstrecke `routes[].legs[].incidents` in Mapbox-Form: `id`, `type`, `description` (Meldungstext unverändert), `sub_type` (Art laut Quelle), `geometry_index_start` und `geometry_index_end` (Stützpunkte der Teilstrecke), `closed`, `impact`, `mandatory`, `start_time`, `end_time`, `creation_time`, `affected_road_names`. Berücksichtigt werden aktive Meldungen in Fahrtrichtung bis 25 m neben der Route. `incidents=false` schaltet das ab. Quellenangabe in `incidents_attribution`.

## Wechselverkehrszeichen

**GET https://api.mapx.at/v1/traffic/signs**

Aktueller Zustand der ASFINAG-Wechselverkehrszeichen: `speed_kmh` (angezeigtes Tempolimit), `pictogram`, `text` (Zusatztexte), `image` (Bild der ASFINAG), `working`. Die Quelle nennt eine Position nur bei einem kleinen Teil der Anlagen (rund 40 von 5300); die übrigen verweisen auf die Anlagentabelle der ASFINAG und erscheinen nur in `total_signs`. Optional `bbox`.

## Status

**GET https://api.mapx.at/v1/traffic/status**

Zeigt Stand der Daten (`fetched`), Zahl der Meldungen und Abschnitte sowie die im Router gesperrten und verlangsamten Kanten.
# Routing

Directions, Isochrone, Matrix und Map Matching auf dem GIP.at-Netz für Fuß, Rad und Auto.

Grundlage ist das österreichische Verkehrsnetz [GIP.at](https://www.gip.at) (CC BY 4.0). Die Endpunkte folgen der Mapbox-Form, `/v1/...` mit dem Profil im Pfad, und brauchen kein Zugangstoken. Profile: `walking`, `cycling`, `driving`, `driving-traffic` (Auto mit aktueller Verkehrslage und Sperren, siehe [Verkehr](/api/traffic/); ein Präfix `mapbox/` wird akzeptiert). Koordinaten stehen als `lon,lat` im Pfad, mehrere Punkte mit `;` getrennt.

## Directions

**GET https://api.mapx.at/v1/directions/{profile}/{coordinates}**

Gleichwertig: `GET /v1/directions?profile=…&coordinates=…`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Verkehrsmittel", values: [{ v: "walking", d: "zu Fuß" }, { v: "cycling", d: "Rad" }, { v: "cycling-mtb", d: "Mountainbike" }, { v: "driving", d: "Auto" }, { v: "driving-traffic", d: "Auto mit Verkehrslage und Sperren" }] },
  { name: "coordinates", type: "string", required: true, desc: "Start, optionale Zwischenpunkte und Ziel als <code>lon,lat;lon,lat;…</code>." },
  { name: "geometries", type: "string", desc: "Format der Geometrie, <code>geojson</code> liefert eine GeoJSON-Linie." },
  { name: "overview", type: "string", desc: "Geometrie: <code>simplified</code> (Standard, vereinfacht wie bei Mapbox), <code>full</code> (alle Stützpunkte) oder <code>false</code> (keine Geometrie)." },
  { name: "steps", type: "boolean", default: "false", desc: "Mit <code>true</code> Abbiegehinweise in <code>legs[].steps</code>." },
  { name: "language", type: "string", desc: "Sprache der Hinweise, zum Beispiel <code>de</code>." },
  { name: "alternatives", type: "boolean | integer", default: "false", desc: "<code>true</code> liefert bis zu zwei weitere Routen in <code>routes[]</code>, eine Zahl von 0 bis 3 begrenzt die Anzahl. Nur mit genau zwei Koordinaten." },
  { name: "annotations", type: "string", desc: "<code>duration</code>, <code>distance</code>, <code>speed</code>, <code>maxspeed</code>, <code>elevation</code>, kommagetrennt. Liefert <code>legs[].annotation</code> mit einem Wert je Segment der Geometrie (Sekunden, Meter, Meter pro Sekunde). <code>maxspeed</code>: <code>{speed, unit: \"km/h\"}</code> oder <code>{unknown: true}</code> (nur Pkw, wo die GIP ein Limit führt). <code>elevation</code>: mittlere Höhe des Segments in Metern, <code>null</code> wo unbekannt." },
  { name: "incidents", type: "boolean", default: "true bei driving-traffic", desc: "Verkehrsmeldungen entlang der Route in <code>legs[].incidents</code> (nur Pkw), siehe <a href=\"/api/traffic/\">Verkehr</a>." },
  { name: "radiuses", type: "string", desc: "Suchradius für die Bindung ans Netz je Koordinate in Metern, mit <code>;</code> getrennt, oder <code>unlimited</code>." },
  { name: "approaches", type: "string", desc: "<code>unrestricted</code> oder <code>curb</code> je Koordinate. Wird geprüft, vom Router aber noch nicht ausgewertet." },
  { name: "bearings", type: "string", desc: "<code>Grad,Toleranz</code> je Koordinate, mit <code>;</code> getrennt (leer lässt einen Punkt frei). Wird geprüft, vom Router aber noch nicht ausgewertet." },
  { name: "weight", type: "number", default: "40", desc: "Nur <code>driving-truck</code>: Gesamtgewicht in Tonnen (0,5 bis 60)." },
  { name: "height", type: "number", default: "4", desc: "Nur <code>driving-truck</code>: Fahrzeughöhe in Metern (1,5 bis 5)." },
  { name: "width", type: "number", default: "2,55", desc: "Nur <code>driving-truck</code>: Fahrzeugbreite in Metern (1 bis 3,5)." },
  { name: "length", type: "number", desc: "Nur <code>driving-truck</code>: Fahrzeuglänge in Metern (2 bis 40). Ohne Angabe werden Längenbeschränkungen nicht geprüft." },
  { name: "truck_type", type: "string", default: "semi", desc: "Nur <code>driving-truck</code>: <code>semi</code> (Sattelzug), <code>rigid</code> (Lkw ohne Anhänger) oder <code>trailer</code> (Lkw-Zug)." },
  { name: "cycling_surface", type: "string", default: "any", desc: "Nur <code>cycling</code>. <code>any</code> bewertet unbefestigte und Gelände-Abschnitte schlechter, ohne sie zu sperren. <code>paved</code> (ganz sperren) ist derzeit nicht verfügbar und liefert 400." },
]} />

Antwort: `code` (`Ok`) und `routes[]` mit `distance` (Meter), `duration` (Sekunden), `legs` und `geometry`.

**GIP-Attribute in den Schritten:** `steps[].ref` (zum Beispiel `A1`, `B83`), `steps[].destinations` (echte Wegweiserziele der GIP, Fernziele zuerst), `steps[].milestone` (`{ref, km}`, Straßenkilometer am Schrittbeginn, nur kilometrierte Straßen mit Nummer), `steps[].maxspeed` (nur Pkw), `ascent_m`/`descent_m` und für `cycling`/`walking` `grade_pct` (mittlere Steigung). Für `cycling` und `cycling-mtb` zusätzlich `steps[].bike_infrastructure` (`cycleway`, `lane`, `shared` oder `none`, was auf dem größten Teil des Schritts gilt) und `steps[].surface` (`paved`, `cobblestone`, `unpaved`, `offroad` oder `unknown`).

**Radprofile:** `cycling` bevorzugt Radwege, Radfahrstreifen, Fahrradstraßen und beschilderte Radrouten (GIP BIKE_HIKE), bewertet unbefestigte Abschnitte und Steigungen schlechter und meidet Hauptstraßen. `cycling-mtb` bevorzugt Forstwege und Singletrails, die MTB-Schwierigkeit der GIP fließt ein. Die Höhen im Inneren von Brücken und Tunneln werden nicht aus dem Geländemodell übernommen. `routes[].summary` enthält `ascent_m`, `descent_m`, `max_grade_pct`, `min_elevation_m`, `max_elevation_m`; Höhen stammen aus der GIP und werden über 200 m geglättet.

**Alternativen** entstehen über seitlich versetzte Zwischenpunkte: Der Router rechnet Start → Zwischenpunkt → Ziel für zehn Punkte entlang der schnellsten Route und behält Wege, die höchstens 1,4-mal so lange dauern, keine Wende enthalten und sich zu höchstens 70 Prozent mit der Hauptroute und untereinander decken. `routes[0]` ist immer die schnellste Route, die übrigen folgen nach Dauer. Die Anfrage kostet ein Mehrfaches einer einfachen Route im Rate-Limit.

**Ausschlüsse:** `exclude=motorway` und `exclude=toll` (auch kommagetrennt, nur Profil `driving`). `motorway` sperrt Autobahnen (GIP-Kategorie A), Schnellstraßen bleiben erlaubt. `toll` sperrt die Streckenmaut-Abschnitte (A9 Gleinalm und Bosruck, A10 Tauern und Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg); vignettenpflichtige Strecken bleiben erlaubt, dafür gibt es `motorway`. Andere Werte ergeben `400`. Liegt Start oder Ziel auf einem gesperrten Abschnitt, findet der Router keine Route.

**Maut in der Antwort:** Jede Pkw-Route (`driving`) enthält `routes[].toll`: `vignette_required` (mindestens 300 m auf vignettenpflichtigen Abschnitten), `vignette_m` (Meter auf vignettenpflichtigen Abschnitten), `vignette_products[]` (Pkw-Vignetten mit `name` und `amount` in Euro), `special_toll[]` (Mautstellen der Streckenmaut am Weg mit `name`, `amount`, `location`), `special_toll_total` und `currency`. Eine Mautstelle zählt je Fahrt einmal, auch wenn mehrere Richtungsabschnitte in den Daten stehen. Quelle: ASFINAG, Mautinformationen, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Die Angaben gelten für Pkw ohne Anhänger; Preise stehen im Datensatz für das Jahr 2026. **Streckenmaut-Links:** Zusätzlich liefert jede Route `routes[].toll_road` (`true`, wenn sie über Links mit dem GIP-Attribut TOLL führt, also Sondermaut-Abschnitte wie A13 Brenner) und `toll_road_m` (Meter darauf). Mit `steps=true` enthält jeder Schritt `intersections[0].classes` im Mapbox-Stil: `toll` (Schritt berührt Streckenmaut-Links), `motorway`, `ferry`. Vignettenstrecken sind nicht `toll`; sie stehen in `toll.vignette_*`. Der Preis der Sondermaut steht in `toll.special_toll[]`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/cycling/15.33,48.23;15.34,48.24?geometries=geojson&overview=full"`} />

Live: [Route](/beispiele/route/).

## Lkw-Routing

Das Profil `driving-truck` (`/v1/directions/driving-truck/{coordinates}`, ebenso `isochrone` und `matrix`) routet Fahrzeuge über 3,5 t auf den GIP-Lkw-Daten (`LINK_2_TRUCK` und die `max_*`-Spalten des GeoPackage; bei Abweichung gilt die strengere Angabe). Ohne Parameter gilt: 40 t, 4 m hoch, 2,55 m breit, Sattelzug, Länge ungeprüft.

- **Gesperrt** wird nach Fahrzeug: Gewichtsklassen-Verbote (über 3,5 / 7,5 / 16 t, je Richtung), Sattelzug- und Lkw-Zug-Verbote sowie Beschränkungen für Höhe, Breite, Gewicht und Länge. Straßen mit „Durchfahrt für Lkw verboten, Anlieger frei“ bleiben mit hohem Aufschlag offen und erscheinen als Hinweis `destination_only`.
- **Bezirks-Fahrverbote** (958 Verordnungen als Freitext mit Zeiten und Ausnahmen) werden nicht geroutet. Führt die Route über eine betroffene Straße, steht in `steps[].notices[]` ein Eintrag `{type: "truck_restriction", id, text}` mit dem Verordnungstext; Zeiten und Ausnahmen sind dort nachzulesen.
- **Tempo:** höchstens 80 km/h auf Autobahnen, 70 km/h auf Freilandstraßen, 50 km/h im Ortsgebiet (oder die GIP-Geschwindigkeit, wenn sie niedriger ist).
- **Maut:** Die GO-Maut gilt auf allen Autobahnen und Schnellstraßen für Fahrzeuge über 3,5 t. `routes[].toll.truck_go_km` nennt die Kilometer, je Schritt steht `truck_go_m`. Der Tarif (Achszahl, Emissionsklasse) wird nicht berechnet; Sondermautstrecken (Brenner, Tauern, Arlberg u. a.) kommen hinzu.
- `routes[].truck` gibt das verwendete Fahrzeug zurück.
- **Grenzen:** Das Standardfahrzeug läuft auf Contraction Hierarchies (Millisekunden). Andere Maße brauchen eine berechnete Überlagerung (einmalig etwa 50 ms, danach im Zwischenspeicher) und eine Suche ohne Contraction Hierarchies, eine Anfrage dauert dann etwa 0,1 bis 0,5 s. Das gilt auch für `isochrone` und `matrix`. Achslast gibt es nicht, die GIP-OGD enthält dazu keine Daten. Start und Ziel rasten im Netz des Standardfahrzeugs ein; ein Ort, den nur ein leichterer Lkw erreicht, findet keine Route.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/driving-truck/14.29,48.30;16.37,48.21?steps=true&weight=26&height=3.6&width=2.55&length=16.5"`} />


## Isochrone

**GET https://api.mapx.at/v1/isochrone/{profile}/{coordinate}**

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "<code>walking</code>, <code>cycling</code> oder <code>driving</code>." },
  { name: "coordinate", type: "string", required: true, desc: "Startpunkt <code>lon,lat</code>." },
  { name: "contours_minutes", type: "string", default: "5,10,15", desc: "Zeitstufen in Minuten, kommagetrennt, bis zu vier Werte." },
  { name: "contours_meters", type: "string", desc: "Alternativ Distanzstufen in Metern." },
]} />

Antwort: `FeatureCollection`, je Stufe ein `MultiPolygon` (größte zuerst) mit `contour`, `unit`, `metric` und `area_km2`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/isochrone/walking/15.33,48.23?contours_minutes=5,10,15"`} />

Live: [Erreichbarkeit](/beispiele/isochrone/).

## Matrix

**GET https://api.mapx.at/v1/matrix/{profile}/{coordinates}**

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Wie oben." },
  { name: "coordinates", type: "string", required: true, desc: "Alle Orte als <code>lon,lat;lon,lat;…</code>." },
  { name: "sources", type: "string", desc: "Indizes der Quellen, mit <code>;</code> getrennt; ohne Angabe alle." },
  { name: "destinations", type: "string", desc: "Indizes der Ziele, mit <code>;</code> getrennt; ohne Angabe alle." },
  { name: "annotations", type: "string", desc: "<code>duration</code>, <code>distance</code> oder beides, kommagetrennt." },
]} />

Antwort: `durations` (Sekunden) und bei Bedarf `distances` (Meter) als Matrix, `null` heißt nicht routbar; dazu `sources` und `destinations` mit gebundenem Punkt.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/matrix/driving/15.33,48.23;15.34,48.24;15.3,48.2"`} />

Live: [Fahrzeitmatrix](/beispiele/matrix/).

## Map Matching

**GET https://api.mapx.at/v1/matching/{profile}/{coordinates}**

Ordnet eine GPS-Spur dem Wegenetz zu.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Wie oben." },
  { name: "coordinates", type: "string", required: true, desc: "Spur als <code>lon,lat;lon,lat;…</code>." },
  { name: "timestamps", type: "string", desc: "Sekunden je Punkt, mit <code>;</code> getrennt; verbessert die Zuordnung." },
  { name: "radiuses", type: "string", desc: "Suchradius je Punkt in Metern, mit <code>;</code> getrennt." },
  { name: "geometries", type: "string", desc: "Format der Geometrie, <code>geojson</code> für GeoJSON." },
]} />

Antwort: `matchings[]` mit `geometry`, `distance`, `duration` und `confidence` sowie `tracepoints` (gebundener Punkt oder `null`).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/matching/driving/15.33019,48.2299;15.3295,48.229?geometries=geojson"`} />

## Optimierte Reihenfolge

**GET https://api.mapx.at/v1/optimized-trips/{profile}/{coordinates}**

Findet die beste Reihenfolge für 2 bis 25 Punkte (Handlungsreisender, Mapbox Optimization API v1) und liefert die Route in dieser Reihenfolge. Bis 12 Punkte wird exakt gerechnet, darüber mit Nearest-Neighbor, 2-opt und Or-opt auf der Fahrzeitmatrix.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Wie oben." },
  { name: "coordinates", type: "string", required: true, desc: "2 bis 25 Orte als <code>lon,lat;lon,lat;…</code>." },
  { name: "source", type: "string", default: "any", desc: "<code>first</code>: Der erste Punkt ist der Start. <code>any</code>: beliebiger Start." },
  { name: "destination", type: "string", default: "any", desc: "<code>last</code>: Der letzte Punkt ist das Ende. <code>any</code>: beliebiges Ende." },
  { name: "roundtrip", type: "boolean", default: "true", desc: "Mit <code>true</code> kehrt die Fahrt zum Start zurück. Mit <code>destination=last</code> ist <code>source=first</code> Pflicht, dann wird der Weg vom ersten zum letzten Punkt gesucht und die Rückfahrt zum Start kommt dazu." },
  { name: "steps", type: "boolean", default: "false", desc: "Abbiegehinweise in <code>legs[].steps</code>." },
  { name: "geometries", type: "string", default: "geojson", desc: "<code>geojson</code>, <code>polyline</code> oder <code>polyline6</code>." },
  { name: "overview", type: "string", desc: "<code>simplified</code> (Standard), <code>full</code> oder <code>false</code> (ohne Geometrie)." },
  { name: "language", type: "string", default: "de", desc: "<code>de</code> oder <code>en</code>." },
]} />

Antwort: `trips[]` mit einer Route (`distance`, `duration`, `legs`, `geometry`) und `waypoints[]` in der Reihenfolge der Eingabe. Jeder Eintrag hat `waypoint_index` (Position in der Fahrt) und `trips_index` (immer 0). Sind Punkte nicht miteinander verbunden, kommt `422`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/optimized-trips/driving/15.3338,48.2279;15.2065,48.2111;15.0864,48.1696;15.3969,48.2339?source=first&roundtrip=true"`} />

Die Anfrage zählt wie Matrix und Isochrone zu den aufwendigen.

Für Touren mit mehreren Fahrzeugen, Kapazitäten und Zeitfenstern gibt es [Tourenplanung](/api/optimization/).

## Fehler und Limits

Fehler kommen als `application/problem+json` mit passendem HTTP-Status (`400` ungültige Eingabe, `404` keine Route oder kein Netz in der Nähe, `429` Limit). Isochrone und Matrix haben enge Limits. Allgemeines: [Fehler und Limits](/api/fehler/).
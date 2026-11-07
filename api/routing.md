# Routing

Directions, Isochrone, Matrix und Map Matching auf dem GIP.at-Netz für Fuß, Rad und Auto.

Grundlage ist das österreichische Verkehrsnetz [GIP.at](https://www.gip.at) (CC BY 4.0). Die Endpunkte folgen der Mapbox-Form, `/v1/...` mit dem Profil im Pfad, und brauchen kein Zugangstoken. Profile: `walking`, `cycling`, `driving`, `driving-traffic` (Auto mit aktueller Verkehrslage und Sperren, siehe [Verkehr](/api/traffic/); ein Präfix `mapbox/` wird akzeptiert). Koordinaten stehen als `lon,lat` im Pfad, mehrere Punkte mit `;` getrennt.

## Directions

**GET https://api.mapx.at/v1/directions/{profile}/{coordinates}**

Gleichwertig: `GET /v1/directions?profile=…&coordinates=…`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Verkehrsmittel", values: [{ v: "walking", d: "zu Fuß" }, { v: "cycling", d: "Rad" }, { v: "driving", d: "Auto" }, { v: "driving-traffic", d: "Auto mit Verkehrslage und Sperren" }] },
  { name: "coordinates", type: "string", required: true, desc: "Start, optionale Zwischenpunkte und Ziel als <code>lon,lat;lon,lat;…</code>." },
  { name: "geometries", type: "string", desc: "Format der Geometrie, <code>geojson</code> liefert eine GeoJSON-Linie." },
  { name: "overview", type: "string", desc: "Detailgrad der Geometrie, zum Beispiel <code>full</code>." },
  { name: "steps", type: "boolean", default: "false", desc: "Mit <code>true</code> Abbiegehinweise in <code>legs[].steps</code>." },
  { name: "language", type: "string", desc: "Sprache der Hinweise, zum Beispiel <code>de</code>." },
  { name: "alternatives", type: "boolean | integer", default: "false", desc: "<code>true</code> liefert bis zu zwei weitere Routen in <code>routes[]</code>, eine Zahl von 0 bis 3 begrenzt die Anzahl. Nur mit genau zwei Koordinaten." },
  { name: "annotations", type: "string", desc: "<code>duration</code>, <code>distance</code>, <code>speed</code>, kommagetrennt. Liefert <code>legs[].annotation</code> mit einem Wert je Segment der Geometrie (Sekunden, Meter, Meter pro Sekunde)." },
  { name: "incidents", type: "boolean", default: "true bei driving-traffic", desc: "Verkehrsmeldungen entlang der Route in <code>legs[].incidents</code> (nur Pkw), siehe <a href=\"/api/traffic/\">Verkehr</a>." },
  { name: "radiuses", type: "string", desc: "Suchradius für die Bindung ans Netz je Koordinate in Metern, mit <code>;</code> getrennt, oder <code>unlimited</code>." },
  { name: "approaches", type: "string", desc: "<code>unrestricted</code> oder <code>curb</code> je Koordinate. Wird geprüft, vom Router aber noch nicht ausgewertet." },
  { name: "bearings", type: "string", desc: "<code>Grad,Toleranz</code> je Koordinate, mit <code>;</code> getrennt (leer lässt einen Punkt frei). Wird geprüft, vom Router aber noch nicht ausgewertet." },
]} />

Antwort: `code` (`Ok`) und `routes[]` mit `distance` (Meter), `duration` (Sekunden), `legs` und `geometry`.

**Alternativen** entstehen über seitlich versetzte Zwischenpunkte: Der Router rechnet Start → Zwischenpunkt → Ziel für zehn Punkte entlang der schnellsten Route und behält Wege, die höchstens 1,4-mal so lange dauern, keine Wende enthalten und sich zu höchstens 70 Prozent mit der Hauptroute und untereinander decken. `routes[0]` ist immer die schnellste Route, die übrigen folgen nach Dauer. Die Anfrage kostet ein Mehrfaches einer einfachen Route im Rate-Limit.

**Ausschlüsse:** `exclude=motorway` und `exclude=toll` (auch kommagetrennt, nur Profil `driving`). `motorway` sperrt Autobahnen (GIP-Kategorie A), Schnellstraßen bleiben erlaubt. `toll` sperrt die Streckenmaut-Abschnitte (A9 Gleinalm und Bosruck, A10 Tauern und Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg); vignettenpflichtige Strecken bleiben erlaubt, dafür gibt es `motorway`. Andere Werte ergeben `400`. Liegt Start oder Ziel auf einem gesperrten Abschnitt, findet der Router keine Route.

**Maut in der Antwort:** Jede Pkw-Route (`driving`, auch `driving-ev`) enthält `routes[].toll`: `vignette_required` (mindestens 300 m auf vignettenpflichtigen Abschnitten), `vignette_m` (Meter auf vignettenpflichtigen Abschnitten), `vignette_products[]` (Pkw-Vignetten mit `name` und `amount` in Euro), `special_toll[]` (Mautstellen der Streckenmaut am Weg mit `name`, `amount`, `location`), `special_toll_total` und `currency`. Eine Mautstelle zählt je Fahrt einmal, auch wenn mehrere Richtungsabschnitte in den Daten stehen. Quelle: ASFINAG, Mautinformationen, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Die Angaben gelten für Pkw ohne Anhänger; Preise stehen im Datensatz für das Jahr 2026. **Streckenmaut-Links:** Zusätzlich liefert jede Route `routes[].toll_road` (`true`, wenn sie über Links mit dem GIP-Attribut TOLL führt, also Sondermaut-Abschnitte wie A13 Brenner) und `toll_road_m` (Meter darauf). Mit `steps=true` enthält jeder Schritt `intersections[0].classes` im Mapbox-Stil: `toll` (Schritt berührt Streckenmaut-Links), `motorway`, `ferry`. Vignettenstrecken sind nicht `toll`; sie stehen in `toll.vignette_*`. Der Preis der Sondermaut steht in `toll.special_toll[]`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/cycling/15.33,48.23;15.34,48.24?geometries=geojson&overview=full"`} />

Live: [Route](/beispiele/route/).

## E-Auto-Routing

> Solange die Nutzungsbedingungen der E-Control-Ladedaten nicht geklärt sind, antwortet der Endpunkt mit `404`.


**GET https://api.mapx.at/v1/directions/driving-ev/{coordinates}**

Genau Start und Ziel. Die Pkw-Route wird berechnet, bei zu geringer Reichweite wählt der Dienst Ladestationen entlang der Route ([Daten](/daten/ladestationen/)) und routet über sie. Reichweitenmodell linear: Verbrauch pro Kilometer konstant, Start mit voller Batterie, Laden bis `ev_charge_to`, Ankunft nie unter `ev_reserve`.

<ParamTable params={[
  { name: "ev_range_km", type: "number", desc: "Pflicht. Reichweite bei voller Batterie in km (1 bis 1500)." },
  { name: "ev_charge_kw_min", type: "number", default: "50", desc: "Mindest-Ladeleistung der Stationen in kW." },
  { name: "ev_battery_kwh", type: "number", default: "60", desc: "Batteriegröße für die Ladezeit." },
  { name: "ev_charge_kw_max", type: "number", default: "100", desc: "Höchste Ladeleistung des Fahrzeugs in kW." },
  { name: "ev_charge_to", type: "number", default: "0.8", desc: "Ladeziel als Anteil der Batterie." },
  { name: "ev_reserve", type: "number", default: "0.1", desc: "Reserve bei Ankunft als Anteil der Batterie." },
  { name: "steps, language", type: "", desc: "wie bei Directions." },
]} />

Die Antwort ist eine Directions-Antwort, `legs[]` endet jeweils an einer Ladestation. `routes[0].ev` enthält `stops[]` (`id`, `name`, `location`, `max_kw`, `along_m`, `offset_m`, `soc_arrival`, `soc_departure`, `charge_s`), `drive_s`, `charge_s` und `total_s`. `duration` der Route umfasst nur die Fahrzeit, `ev.total_s` zusätzlich das Laden. Findet sich auf einer Etappe keine Station, antwortet der Dienst mit `400` und nennt den Streckenabschnitt. Ladestationen mit mehr als 3 km Abstand zur Route werden nicht verwendet. Datenquelle: E-Control Ladestellenverzeichnis.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/driving-ev/16.37,48.21;11.40,47.27?ev_range_km=300&ev_charge_kw_min=100"`} />

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
  { name: "overview", type: "string", desc: "<code>false</code> lässt die Geometrie weg." },
  { name: "language", type: "string", default: "de", desc: "<code>de</code> oder <code>en</code>." },
]} />

Antwort: `trips[]` mit einer Route (`distance`, `duration`, `legs`, `geometry`) und `waypoints[]` in der Reihenfolge der Eingabe. Jeder Eintrag hat `waypoint_index` (Position in der Fahrt) und `trips_index` (immer 0). Sind Punkte nicht miteinander verbunden, kommt `422`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/optimized-trips/driving/15.3338,48.2279;15.2065,48.2111;15.0864,48.1696;15.3969,48.2339?source=first&roundtrip=true"`} />

Die Anfrage zählt wie Matrix und Isochrone zu den aufwendigen.

Für Touren mit mehreren Fahrzeugen, Kapazitäten und Zeitfenstern gibt es [Tourenplanung](/api/optimization/).

## Fehler und Limits

Fehler kommen als `application/problem+json` mit passendem HTTP-Status (`400` ungültige Eingabe, `404` keine Route oder kein Netz in der Nähe, `429` Limit). Isochrone und Matrix haben enge Limits. Allgemeines: [Fehler und Limits](/api/fehler/).
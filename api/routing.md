# Routing

Directions, Isochrone, Matrix und Map Matching auf dem GIP.at-Netz für Fuß, Rad und Auto.

Grundlage ist das österreichische Verkehrsnetz [GIP.at](https://www.gip.at) (CC BY 4.0). Die Endpunkte folgen der Mapbox-Form, `/v1/...` mit dem Profil im Pfad, und brauchen kein Zugangstoken. Profile: `walking`, `cycling`, `driving` (ein Präfix `mapbox/` wird akzeptiert). Koordinaten stehen als `lon,lat` im Pfad, mehrere Punkte mit `;` getrennt.

## Directions

**GET https://api.mapx.at/v1/directions/{profile}/{coordinates}**

Gleichwertig: `GET /v1/directions?profile=…&coordinates=…`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Verkehrsmittel", values: [{ v: "walking", d: "zu Fuß" }, { v: "cycling", d: "Rad" }, { v: "driving", d: "Auto" }] },
  { name: "coordinates", type: "string", required: true, desc: "Start, optionale Zwischenpunkte und Ziel als <code>lon,lat;lon,lat;…</code>." },
  { name: "geometries", type: "string", desc: "Format der Geometrie, <code>geojson</code> liefert eine GeoJSON-Linie." },
  { name: "overview", type: "string", desc: "Detailgrad der Geometrie, zum Beispiel <code>full</code>." },
  { name: "steps", type: "boolean", default: "false", desc: "Mit <code>true</code> Abbiegehinweise in <code>legs[].steps</code>." },
  { name: "language", type: "string", desc: "Sprache der Hinweise, zum Beispiel <code>de</code>." },
]} />

Antwort: `code` (`Ok`) und `routes[]` mit `distance` (Meter), `duration` (Sekunden), `legs` und `geometry`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/cycling/15.33,48.23;15.34,48.24?geometries=geojson&overview=full"`} />

Live: [Route](/beispiele/route/).

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

## Fehler und Limits

Fehler kommen als `application/problem+json` mit passendem HTTP-Status (`400` ungültige Eingabe, `404` keine Route oder kein Netz in der Nähe, `429` Limit). Isochrone und Matrix haben enge Limits. Allgemeines: [Fehler und Limits](/api/fehler/).
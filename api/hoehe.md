# Höhe und Profil

Höhe für Punkte oder entlang eines Pfads aus DTM oder DSM, GeoJSON-Antwort, mit Testkonsole.

Höhe für einen Punkt, mehrere Punkte oder einen Pfad. Maschinenlesbar: [openapi.yaml](/openapi.yaml), live [/v1/openapi.json](https://api.mapx.at/v1/openapi.json).

<Endpoint url="https://api.mapx.at/v1/elevation" />

<ParamTable params={[
  { name: "points", type: "string", desc: "<code>lon,lat;lon,lat;...</code> (WGS84, Länge vor Breite)" },
  { name: "path", type: "string", desc: "<code>lon,lat;lon,lat;...</code>, Polylinie (2 bis 512 Stützpunkte); zusammen mit <code>samples</code> ein Profil entlang des Pfads" },
  { name: "samples", type: "integer", desc: "Anzahl gleichmäßig nach Länge (Großkreis auf der Kugel) verteilter Messpunkte entlang <code>path</code>, 2 bis 1024, Standard 100" },
  { name: "model", type: "string", desc: "<code>dtm</code> (Standard) oder <code>dsm</code>" },
]} />

`points` und `path` schließen sich aus; `points` nimmt höchstens 512 Punkte. Die Höhe wird bilinear aus den z16-Kacheln (0,1-m-Raster) interpoliert.

### Ausprobieren

Die Konsole schickt eine echte Anfrage an den Dienst.

<Testkonsole />

### Beispiel: Punkt

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation?points=12.694,47.074&model=dtm"`} />

Antwort: GeoJSON `FeatureCollection`. Jedes Feature ist ein Punkt (Koordinaten `[lon, lat]`) mit den Eigenschaften `elevation` (m), `resolution_m` (Rasterweite der Quelle, 1 m), `model` und `stichtag`; bei `path` zusätzlich `distance_m` (Strecke vom Pfadanfang). Außerhalb der Daten ist `elevation` `null` (Antwort bleibt `200`).

<Code lang="json" code={`{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "geometry": { "type": "Point", "coordinates": [12.694, 47.074] },
    "properties": { "elevation": 0.0, "resolution_m": 1.0, "model": "dtm", "stichtag": "2025-09-15" }
  }]
}`} />

Der Wert von `elevation` im Beispiel zeigt nur die Form der Antwort.

### Beispiel: Profil

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50"`} />

Antwort wie oben, ein Feature je Messpunkt in Pfadreihenfolge.

## Fehler

`application/problem+json` mit `type`, `title`, `status` und `detail`:

<Code lang="json" code={`{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "'12.5': erwartet lon,lat"
}`} />

Typische Fälle: `400` (Parameter fehlen oder sind fehlerhaft), `404` (Tileset oder Kachel unbekannt), `429` (zu viele Anfragen, mit `Retry-After`), `503` (Dienst ausgelastet oder Höhenmodell nicht geladen).
`type` ist `about:blank`; `title` ist der Statustext, `detail` die Erklärung.

Mehr: [Fehler, Limits und Betrieb](/api/fehler/).
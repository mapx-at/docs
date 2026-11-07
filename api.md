# API

Kachel-, TileJSON-, XYZ- und Höhen-Endpunkte von mapx. Offen, ohne Schlüssel, Fehler als RFC 9457.

<Aside type="caution" title="Status">
Die Endpunkte unter `/v1/` laufen im Dienst `mapx-api`; ob sie öffentlich schon antworten, steht im [Changelog](/changelog/).
Ohne Abhängigkeit davon funktioniert immer der Weg über die PMTiles-Datei ([Abschnitt tiles](#tiles)).
</Aside>

Gemeinsam für alle Endpunkte: kein API-Schlüssel, CORS offen (`Access-Control-Allow-Origin: *`), Fehler als `application/problem+json` nach
[RFC 9457](https://www.rfc-editor.org/rfc/rfc9457). Der Dienst begrenzt Anfragen je IP (Standard: 50 pro Sekunde, Spitze 300; Höhenabfragen zählen je 32 Punkte zusätzlich eins) und antwortet darüber mit `429` und `Retry-After`.

## tiles

<Endpoint url="https://tiles.mapx.at/<datei>.pmtiles" />

Die ganze PMTiles-Datei mit HTTP-Range. Das ist der Basisweg, MapLibre liest damit über das `pmtiles://`-Protokoll.
Dateinamen enthalten Stichtag und Hash und ändern sich nie. Aktuelle Namen: [DTM](/daten/dtm/#1-kennwerte).

<Code lang="bash" code={`curl -s -r 0-126 https://tiles.mapx.at/<datei>.pmtiles | xxd | head`} />

## tilejson

<Endpoint url="https://tiles.mapx.at/v1/{tileset}.json" />

Zum Beispiel `/v1/dtm.json` und `/v1/dsm.json`. `{tileset}` ist `dtm`, `dsm` oder der Tileset-Name aus dem Manifest
(`relief-terrainrgb`, `relief-dsm-terrainrgb`, `mapx-basis`, `hoehenlinien`, …); Vektor-Tilesets liefern zusätzlich `vector_layers`. Die Antwort trägt einen `ETag` und `Cache-Control`.

Antwort: [TileJSON 3.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0) mit `tiles`, `minzoom`, `maxzoom`, `bounds`, `attribution`, `tileSize`, `encoding`
und dem Stichtag. Damit lässt sich ein Tileset per URL in Clients einbinden, ohne den Dateinamen zu kennen.

<Code lang="json" code={`{
  "tilejson": "3.0.0",
  "name": "relief-terrainrgb",
  "tiles": ["https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp"],
  "minzoom": 5, "maxzoom": 16,
  "bounds": [9.5306396, 46.3696741, 17.1661376, 49.0234614],
  "tileSize": 512, "encoding": "mapbox",
  "attribution": "BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0"
}`} />

Das Beispiel zeigt die Form; Zahlen entsprechen den PMTiles-Metadaten, die verbindliche Antwort liefert der Endpunkt.

## xyz

<Endpoint url="https://tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.webp" />

Einzelne Kachel für Clients ohne PMTiles-Unterstützung (Leaflet, OpenLayers, QGIS). Inhalt wie in der PMTiles-Datei: WebP verlustfrei, 512 px, Terrain-RGB.
Die Endung passt zum Kacheltyp (`webp`, `png`, `jpg`, `avif`, bei Vektorkacheln `pbf`), sonst `404`. Vektorkacheln kommen mit `Content-Encoding: gzip`, wenn der Client es annimmt.
Gibt es die Kachel nicht (außerhalb der Daten oder des Zoombereichs), antwortet der Dienst mit `404` als `problem+json` (wie Mapbox, nicht `204`). `ETag` ist der Datei-Hash, `If-None-Match` liefert `304`.

<Code lang="bash" code={`curl -s -o kachel.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp`} />

## elevation

Höhe für einen Punkt, mehrere Punkte oder einen Pfad. Maschinenlesbar: [openapi.yaml](/openapi.yaml).

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

Der Wert von `elevation` im Beispiel ist ein Platzhalter für die Form, kein Messwert.

### Beispiel: Profil

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50"`} />

Antwort wie oben, ein Feature je Messpunkt in Pfadreihenfolge.

### Fehler

`application/problem+json` mit `type`, `title`, `status` und `detail`:

<Code lang="json" code={`{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "'12.5': erwartet lon,lat"
}`} />

Typische Fälle: `400` (Parameter fehlen oder sind fehlerhaft), `404` (Tileset oder Kachel unbekannt), `429` (zu viele Anfragen, mit `Retry-After`), `503` (Dienst ausgelastet oder Höhenmodell nicht geladen).
`type` ist `about:blank`; `title` ist der Statustext, `detail` die Erklärung.

## Fehler

Alle Endpunkte antworten bei Fehlern mit `application/problem+json` (RFC 9457), nicht mit HTML.
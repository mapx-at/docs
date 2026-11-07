# Kacheln (PMTiles und XYZ)

Ganze PMTiles-Datei per HTTP-Range oder einzelne Kacheln über /v1/{tileset}/{z}/{x}/{y}.

Zwei Wege zu denselben Kacheln: die ganze PMTiles-Datei (Basisweg, je Kachel ein Range-Request) oder einzelne XYZ-Kacheln über den Dienst.

## PMTiles

<Endpoint url="https://tiles.mapx.at/<datei>.pmtiles" />

Die ganze PMTiles-Datei mit HTTP-Range. Das ist der Basisweg, MapLibre liest damit über das `pmtiles://`-Protokoll.
Dateinamen enthalten den Stichtag und ändern sich nie. Aktuelle Namen: auf jeder Produktseite unter „Kennwerte“.

<Code lang="bash" code={`curl -s -r 0-126 https://tiles.mapx.at/<datei>.pmtiles | xxd | head`} />

## XYZ

<Endpoint url="https://tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.webp" />

Einzelne Kachel für Clients ohne PMTiles-Unterstützung (Leaflet, OpenLayers, QGIS). Inhalt wie in der PMTiles-Datei: WebP verlustfrei, 512 px, Terrain-RGB.
Die Endung passt zum Kacheltyp (`webp`, `png`, `jpg`, `avif`, bei Vektorkacheln `pbf`), sonst `404`. Vektorkacheln kommen mit `Content-Encoding: gzip`, wenn der Client es annimmt.
Gibt es die Kachel nicht (außerhalb der Daten oder des Zoombereichs), antwortet der Dienst mit `404` als `problem+json` (wie Mapbox, nicht `204`). `ETag` ist der Datei-Hash, `If-None-Match` liefert `304`.

<Code lang="bash" code={`curl -s -o kachel.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp`} />

## Tilesets

| `{tileset}` | Produkt | Typ | Endung |
|---|---|---|---|
| `dtm`, `relief-terrainrgb` | [DTM](/daten/dtm/) | Terrain-RGB | `webp` |
| `dsm`, `relief-dsm-terrainrgb` | [DSM](/daten/dsm/) | Terrain-RGB | `webp` |
| `dop-ortho` | [Orthophoto](/daten/orthophoto/) | Bild | `webp` |
| `mapx-basis` | [Basiskarte](/daten/basiskarte/) | Vektor | `pbf` |
| `hoehenlinien` | [Höhenlinien](/daten/hoehenlinien/) | Vektor | `pbf` |
| `mapx-kataster` | [Kataster](/daten/kataster/) | Vektor | `pbf` |
| `dop-flugblock` | [Flugblöcke](/daten/flugbloecke/) | Vektor | `pbf` |
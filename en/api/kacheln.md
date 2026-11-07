# Tiles (PMTiles and XYZ)

Whole PMTiles file via HTTP range or single tiles via /v1/{tileset}/{z}/{x}/{y}.

Two routes to the same tiles: the whole PMTiles file (base route, one range request per tile) or single XYZ tiles through the service.

## PMTiles

<Endpoint url="https://tiles.mapx.at/<file>.pmtiles" lang="en" />

The whole PMTiles file with HTTP range. This is the base route; MapLibre reads it through the `pmtiles://` protocol.
File names contain the reference date and never change. Current names: on each product page under "Key figures".

<Code lang="bash" code={`curl -s -r 0-1023 -o head.bin https://tiles.mapx.at/<file>.pmtiles`} />

## XYZ

<Endpoint url="https://tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.webp" lang="en" />

Single tile for clients without PMTiles support (Leaflet, OpenLayers, QGIS). Content as in the PMTiles file: lossless WebP, 512 px, Terrain-RGB.
The extension matches the tile type (`webp`, `png`, `jpg`, `avif`, `pbf` for vector tiles), otherwise `404`. Vector tiles come with `Content-Encoding: gzip` if the client accepts it.
If the tile does not exist (outside the data or the zoom range), the service responds with `404` as `problem+json` (like Mapbox, not `204`). `If-None-Match` returns `304`.

<Code lang="bash" code={`curl -s -o kachel.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp`} />

## Tilesets

| `{tileset}` | Product | Type | Extension |
|---|---|---|---|
| `dtm`, `relief-terrainrgb` | [DTM](/en/daten/dtm/) | Terrain-RGB | `webp` |
| `dsm`, `relief-dsm-terrainrgb` | [DSM](/en/daten/dsm/) | Terrain-RGB | `webp` |
| `dop-ortho` | [Orthophoto](/en/daten/orthophoto/) | Image | `webp` |
| `mapx-basis` | [Base map](/en/daten/basiskarte/) | Vector | `pbf` |
| `hoehenlinien` | [Contour lines](/en/daten/hoehenlinien/) | Vector | `pbf` |
| `mapx-kataster` | [Cadastre](/en/daten/kataster/) | Vector | `pbf` |
| `dop-flugblock` | [Flight blocks](/en/daten/flugbloecke/) | Vector | `pbf` |
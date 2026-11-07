# TileJSON

TileJSON 3.0 per tileset at /v1/{tileset}.json, so clients do not need file names.

<Endpoint url="https://tiles.mapx.at/v1/{tileset}.json" lang="en" />

For example `/v1/dtm.json` and `/v1/dsm.json`. `{tileset}` is `dtm`, `dsm` or a tileset name
(`relief-terrainrgb`, `relief-dsm-terrainrgb`, `mapx-basis`, `hoehenlinien`, …); vector tilesets additionally return `vector_layers`. The response carries an `ETag` and `Cache-Control`.

Response: [TileJSON 3.0](https://github.com/mapbox/tilejson-spec/tree/master/3.0.0) with `tiles`, `minzoom`, `maxzoom`, `bounds`, `attribution`, `tileSize`, `encoding`
and the reference date. This lets you add a tileset to clients by URL without knowing the file name.

<Code lang="json" code={`{
  "tilejson": "3.0.0",
  "name": "relief-terrainrgb",
  "tiles": ["https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp"],
  "minzoom": 5, "maxzoom": 16,
  "bounds": [9.5306396, 46.3696741, 17.1661376, 49.0234614],
  "tileSize": 512, "encoding": "mapbox",
  "attribution": "BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0"
}`} />

All tilesets: [Tiles](/en/api/kacheln/#tilesets).
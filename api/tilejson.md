# TileJSON

TileJSON 3.0 je Tileset unter /v1/{tileset}.json, damit Clients ohne Dateinamen auskommen.

<Endpoint url="https://tiles.mapx.at/v1/{tileset}.json" />

Zum Beispiel `/v1/dtm.json` und `/v1/dsm.json`. `{tileset}` ist `dtm`, `dsm` oder der Tileset-Name
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

Das Beispiel zeigt die Form der Antwort.

Alle Tilesets: [Kacheln](/api/kacheln/#tilesets).
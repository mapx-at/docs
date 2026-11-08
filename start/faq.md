# FAQ

## Do I need a token or an account?

Yes, a free account and API token. Register at [account.mapx.at](https://account.mapx.at/), create a token and send it as `Authorization: Bearer` (servers) or `access_token` (browsers, images). CORS is open. See [Access and limits](/start/access-and-limits/).

## Why do I get 401 or 403?

`401` means the token is missing, invalid or expired. `403` means it is blocked, or a public token is used from an origin or app that is not allowed (`origin-not-allowed`). The `code` in the error body names the reason. See [Access and limits](/start/access-and-limits/).

## What does it cost?

Nothing. Rate limits apply per token and zone.

## May I use it commercially?

Yes. The data is licensed [CC BY 4.0](/reference/attribution/): name the source, everything else is free.

## How current is the data?

Each tileset has a source date, shown on its page under [Tilesets](/tilesets/) and in the `stand` field of `GET /v1/tiles`.

## Does a URL change when there is new data?

API URLs (`/v1/tiles/streets/...`, TileJSON) stay the same. PMTiles file names contain the date, so old files stay valid. Use the TileJSON to always get the current file.

## Can I use the maps offline?

Yes. Download a PMTiles file and serve it yourself: [Self-host PMTiles](/guides/self-host-pmtiles/), [QGIS](/guides/qgis/).

## Is there data outside Austria?

Austria only. Orthophoto and base map reach a little into neighboring countries at the border.

## What is the difference between DTM and DSM?

The DTM is bare ground, the DSM includes trees and buildings. Use the DTM for gradients and profiles, the DSM for visibility and building heights. See [Elevation](/guides/elevation/).

## Why do I get 404 for a tile?

The tile lies outside the data area or zoom range, the extension does not match the type (`pbf` vector, `webp` raster) or the tileset name is wrong. Treat 404 as an empty tile.

## Why do I get 429?

You exceeded the rate limit of the zone. Wait `Retry-After` seconds and retry with growing delay. For heavy loads use [PMTiles](/guides/self-host-pmtiles/).

## My points land in the wrong country

Longitude and latitude are swapped. mapx and GeoJSON use `lon,lat`. See [Coordinates](/reference/coordinates/).

## My map is blurry

Raster tiles are 512 px. Set `tileSize: 512` in your client. See [Tile scheme](/reference/tile-scheme/).

## Is the cadastre legally binding?

No. It shows parcel boundaries for orientation and does not replace a land register extract or a survey.

## Which old names still work?

The former German style, tileset and category names work until 2027-05-31. See [Migration](/api/migration/).

## Where do I report a problem?

Write to [support@mapx.at](mailto:support@mapx.at) (technical questions and bugs) or [info@mapx.at](mailto:info@mapx.at) (everything else).


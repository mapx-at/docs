# FAQ: token, licence, offline use

Source: https://docs.mapx.at/start/faq/

> Answers on mapx: do I need a token, what does it cost, commercial use under CC BY 4.0, data freshness, offline maps, data outside Austria, errors 401 and 403.

## Do I need a token or an account?

Yes, a free account and API token. Sign in at [account.mapx.at](https://account.mapx.at/) with your email address (the first time, this creates the account), create a token and send it as `Authorization: Bearer` (servers) or `access_token` (browsers, images). CORS is open. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md).

## Why do I get 401 or 403?

`401` means the token is missing, invalid or expired. `403` means it is blocked, or a public token is used from an origin or app that is not allowed (`origin-not-allowed`), or a public token calls an endpoint that needs a secret token (`pk-not-allowed`). The `code` in the error body (and the `X-Mapx-Reason` header) names the reason. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md) and [error codes](https://docs.mapx.at/api/errors.md#authentication-errors).

## Can I use mapx with Claude, ChatGPT or Cursor?

Yes, through the [MCP server](https://docs.mapx.at/guides/mcp.md): 23 read-only tools for Austrian data. In claude.ai, ChatGPT and Claude Code you only enter `https://mcp.mapx.at/mcp` and sign in with your mapx account. For Cursor, VS Code, Gemini CLI and other clients, create a token of type "AI assistant (MCP)".

## What does it cost?

Nothing. Rate limits apply per token and zone.

## May I use it commercially?

Yes. The data is licensed [CC BY 4.0](https://docs.mapx.at/reference/attribution.md): name the source, everything else is free.

## How current is the data?

Each tileset has a source date, shown on its page under [Tilesets](https://docs.mapx.at/tilesets/index.md) and in the `stand` field of `GET /v1/tiles`.

## Does a URL change when there is new data?

API URLs (`/v1/tiles/streets/...`, TileJSON) stay the same. Styles and TileJSON always point to the current data.

## Can I download the tiles?

No. mapx delivers maps and tiles through the API: style, TileJSON or `{z}/{x}/{y}`, see [Maps](https://docs.mapx.at/guides/maps.md) and [QGIS](https://docs.mapx.at/guides/qgis.md). Whole tileset files for self-hosting are not offered.

## Is there data outside Austria?

Austria only. Orthophoto and base map reach a little into neighboring countries at the border.

## What is the difference between DTM and DSM?

The DTM is bare ground, the DSM includes trees and buildings. Use the DTM for gradients and profiles, the DSM for visibility and building heights. See [Elevation](https://docs.mapx.at/guides/elevation.md).

## Why do I get 404 for a tile?

The tile lies outside the data area or zoom range, the extension does not match the type (`pbf` vector, `webp` raster) or the tileset name is wrong. Treat 404 as an empty tile.

## Why do I get 429?

You exceeded the rate limit of the zone. Wait `Retry-After` seconds and retry with growing delay. For heavy loads, cache tiles in the client.

## My points land in the wrong country

Longitude and latitude are swapped. mapx and GeoJSON use `lon,lat`. See [Coordinates](https://docs.mapx.at/reference/coordinates.md).

## My map is blurry

Raster tiles are 512 px. Set `tileSize: 512` in your client. See [Tile scheme](https://docs.mapx.at/reference/tile-scheme.md).

## Is the cadastre legally binding?

No. It shows parcel boundaries for orientation and does not replace a land register extract or a survey.

## Where do I report a problem?

Write to [support@mapx.at](mailto:support@mapx.at) (technical questions and bugs) or [info@mapx.at](mailto:info@mapx.at) (everything else).

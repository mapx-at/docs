# API overview

All mapx APIs: tiles, TileJSON and elevation. No key, open CORS, errors as RFC 9457.

All APIs follow the same rules:

- **No key** – no sign-up, no token, CORS open for every website.
- **Version in the path** – `/v1/`; existing fields do not change within a version, new ones are only added.
- **Coordinates** – always `lon,lat` (WGS84), several separated by `;`. More: [Coordinates](/en/wissen/koordinaten/).
- **Errors** – `application/problem+json` per [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457).
- **Limits** – per IP address, above that `429` with `Retry-After`.

## Maps

| API | Endpoint | Purpose |
|---|---|---|
| [Tiles](/en/api/kacheln/) | `GET tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}` | raster, elevation and vector tiles |
| [PMTiles](/en/api/kacheln/#pmtiles-file) | `GET tiles.mapx.at/<file>.pmtiles` | whole product as one file, via HTTP range |
| [TileJSON](/en/api/tilejson/) | `GET tiles.mapx.at/v1/{tileset}.json` | product description for map clients |

## Elevation

| API | Endpoint | Purpose |
|---|---|---|
| [Elevation and profile](/en/api/hoehe/) | `GET api.mapx.at/v1/elevation` | elevation for points or along a line |

## Operations

| Endpoint | Purpose |
|---|---|
| `GET api.mapx.at/v1/openapi.json` | OpenAPI description of all endpoints |
| `GET api.mapx.at/health` | health check for monitoring |

        
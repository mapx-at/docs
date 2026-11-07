# API overview

All mapx APIs: tiles, TileJSON, elevation, styles, search, routing and images. No key, open CORS, errors as RFC 9457.

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
| [Elevation and profile](/en/api/hoehe/) | `GET api.mapx.at/v1/elevation`, `/v1/elevation/profile` | elevation for points or along a line |

## Images and SDK

| API | Endpoint | Purpose |
|---|---|---|
| [Static Images](/en/api/static/) | `GET api.mapx.at/v1/static/{style}/{view}/{size}` | map image as PNG or WebP, with overlays |
| [SDK mapx-gl](/en/api/sdk/) | `map.mapx.at/sdk/v1/` | MapLibre add-on with search, style switcher, profile |

## Styles, search and routing

| API | Endpoint | Purpose |
|---|---|---|
| [Styles](/en/api/styles/) | `GET api.mapx.at/v1/styles/{id}` | ready-made MapLibre styles |
| [Geocoding](/en/api/geocoding/) | `GET api.mapx.at/v1/geocode`, `/v1/reverse` | search and address for a point |
| [Directions](/en/api/routing/#directions) | `GET api.mapx.at/v1/directions/{profile}/{coordinates}` | route for walking, cycling, driving |
| [Isochrone](/en/api/routing/#isochrone) | `GET api.mapx.at/v1/isochrone/{profile}/{coordinate}` | reachability areas |
| [Matrix](/en/api/routing/#matrix) | `GET api.mapx.at/v1/matrix/{profile}/{coordinates}` | travel times between many places |
| [Map Matching](/en/api/routing/#map-matching) | `GET api.mapx.at/v1/matching/{profile}/{coordinates}` | GPS trace onto the road network |
| [Optimized trips](/en/api/routing/#optimized-trips) | `GET api.mapx.at/v1/optimized-trips/{profile}/{coordinates}` | best order for up to 25 stops |
| [Vehicle routing](/en/api/optimization/) | `POST api.mapx.at/v1/optimization` | tours for several vehicles with capacities and time windows |

All endpoints run under `https://api.mapx.at/v1/` without an access token. [Examples](/en/beispiele/) show each one live.

## Operations

| Endpoint | Purpose |
|---|---|
| `GET api.mapx.at/v1/openapi.json` | OpenAPI description of all endpoints |
| `GET api.mapx.at/health` | health check for monitoring |

        
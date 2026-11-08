# Access and limits

No key needed. Rate limits per zone, CORS, caching and attribution.

## No key

All endpoints under `https://api.mapx.at/v1/` and all tiles under `https://tiles.mapx.at/` are public. There is no sign-up and no token. CORS allows every origin, so you can call the APIs straight from the browser.

## Rate limits

Limits apply per IP address and zone. Every response carries a `RateLimit-Policy` header, for example `"search";q=10;w=1;burst=30` (10 requests per second plus a burst of 30). Exceeding a limit returns `429` with `Retry-After`.

| Zone | Requests | Used by |
|---|---|---|
| `tiles` | 100/s, burst 200 | tiles, TileJSON, glyphs, sprites |
| `search` | 10/s, burst 30 | geocoding, Search Box, autofill, tilequery, parcel, departures, weather, traffic |
| `route` | 5/s, burst 20 | directions, transit journeys |
| `elevation` | 10/s, burst 20 | elevation and profile |
| `static` | 5/s, burst 60 | Static Tiles, traffic tiles |
| `heavy` | 30/min, burst 5 | static images, isochrone, matrix, matching, optimization, batch geocoding |

For bulk work, download the [PMTiles](/tilesets/) file once instead of requesting millions of tiles. Details and retry advice: [Errors and limits](/api/errors/).

## Attribution

Show the source of the data wherever the map or result is visible. MapLibre adds the text for you when you load a style or TileJSON. See [Attribution](/reference/attribution/).

## Keys later

mapx may add optional access keys in the future. Existing keyless use will keep working within the limits above; this page will describe the change.

## Next

- [First map in 5 minutes](/start/first-map/)
- [Errors and limits](/api/errors/)
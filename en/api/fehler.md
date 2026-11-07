# Errors and limits

Error format per RFC 9457, status codes, rate limits and caching of all mapx APIs.

## Error format

All endpoints answer errors with `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)), never with HTML.

<ParamTable kind="response" params={[
  { name: "type", type: "string", desc: "Error type as a URI: <code>https://docs.mapx.at/api/errors/#&lt;code&gt;</code>" },
  { name: "title", type: "string", desc: "Short summary, English (German with <code>Accept-Language: de</code>)" },
  { name: "status", type: "integer", desc: "HTTP status" },
  { name: "detail", type: "string", desc: "Explanation of what is wrong with the request" },
  { name: "code", type: "string", desc: "Stable error code, see the <a href=\"/en/api/errors/\">error catalog</a>" },
  { name: "param", type: "string", desc: "Offending parameter (where applicable)" },
  { name: "allowed", type: "array", desc: "Accepted values for enumerations (where applicable)" },
]} />

<Code lang="json" code={`{ "type": "https://docs.mapx.at/api/errors/#unknown-path", "title": "Unknown path", "status": 404, "detail": "Unknown path", "code": "unknown-path" }`} />

All codes with meaning and remedy: [error catalog](/en/api/errors/).

## Status codes

| Status | Meaning | What to do |
|---|---|---|
| `200` | Success | |
| `304` | Not modified (with <code>If-None-Match</code>) | use the cached copy |
| `400` | Parameters missing or invalid | read <code>detail</code>, fix the request |
| `404` | unknown product, wrong extension, no tile at this position or zoom | for tiles, treat as an empty tile |
| `422` | request valid but cannot be answered (e.g. no route) | change the points or the time |
| `429` | Rate limit exceeded | wait <code>Retry-After</code> seconds, then retry with growing back-off |
| `503` | Temporarily unavailable | retry later |

## Rate limits

Limits apply per IP address and zone. Every response carries `RateLimit-Policy`, for example `"search";q=10;w=1;burst=30` (10 requests per second plus a burst of 30). Zones: tiles, fonts, sprites 100/s (burst 200); search, geocoding, Search Box, autofill, departures, weather, traffic 10/s (30); routing 5/s (20); elevation 10/s (20); Static Tiles and traffic tiles 5/s (60); expensive endpoints (Static Images, isochrone, matrix, matching, optimized trips, optimization, batch) 30 per minute (3 to 5).

| API | Requests | Per request |
|---|---|---|
| Tiles, TileJSON | 50 per second, bursts up to 300 | one tile |
| PMTiles | as tiles | any range requests |
| Elevation and profile | 10 per second, bursts up to 20; every 32 points count as one more request | up to 512 points, up to 1024 profile samples |

For heavier loads (batch processing, whole regions) use the PMTiles file: download or mirror it once and read it locally.

## Caching

| Header | Value |
|---|---|
| `Cache-Control` | `public, max-age=3600` (tiles, TileJSON, elevation, parcel), `max-age=300` (geocoding, reverse, autofill suggestions, address validation via GET, tileset list), `max-age=0` (routing, matrix, isochrone, POST requests) |
| `ETag`, `Last-Modified` | version; with `If-None-Match` / `If-Modified-Since` → `304` (tiles, styles, TileJSON, OpenAPI) |
| `Server-Timing` | `router`, `search`, `render`, `total` in milliseconds |
| `RateLimit-Policy` | zone and limit of the endpoint |

PMTiles files carry the release date in their name and never change; they may be cached indefinitely.

## Attribution

JSON responses carrying data have a top-level `attribution` field (plain text, no HTML) naming the source to credit when displaying or passing on the data, e.g. `Datenquelle: BEV – DKM, CC BY 4.0` for parcels or `Datenquelle: GIP.at …` for routing, matrix and isochrone. Exceptions: the lists `/v1/styles`, `/v1/fonts`, `/v1/sprites` (arrays); styles credit their sources in `sources`. See [Attribution](/en/anleitungen/attribution/).

## Operations

- `GET https://api.mapx.at/health` – health check for monitoring.
- `GET https://api.mapx.at/v1/openapi.json` – OpenAPI 3.1 description (English, with examples).
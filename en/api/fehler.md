# Errors and limits

Error format per RFC 9457, status codes, rate limits and caching of all mapx APIs.

## Error format

All endpoints answer errors with `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)), never with HTML.

<ParamTable kind="response" params={[
  { name: "type", type: "string", desc: "Error type: <code>about:blank</code>" },
  { name: "title", type: "string", desc: "HTTP status text, e.g. <code>Bad Request</code>" },
  { name: "status", type: "integer", desc: "HTTP status" },
  { name: "detail", type: "string", desc: "Explanation of what is wrong with the request" },
]} />

<Code lang="json" code={`{ "type": "about:blank", "title": "Not Found", "status": 404, "detail": "..." }`} />

## Status codes

| Status | Meaning | What to do |
|---|---|---|
| `200` | Success | |
| `304` | Not modified (with <code>If-None-Match</code>) | use the cached copy |
| `400` | Parameters missing or invalid | read <code>detail</code>, fix the request |
| `404` | unknown product, wrong extension, no tile at this position or zoom | for tiles, treat as an empty tile |
| `429` | Rate limit exceeded | wait <code>Retry-After</code> seconds, then retry with growing back-off |
| `503` | Temporarily unavailable | retry later |

## Rate limits

Limits apply per IP address.

| API | Requests | Per request |
|---|---|---|
| Tiles, TileJSON | 50 per second, bursts up to 300 | one tile |
| PMTiles | as tiles | any range requests |
| Elevation and profile | 10 per second, bursts up to 20; every 32 points count as one more request | up to 512 points, up to 1024 profile samples |

For heavier loads (batch processing, whole regions) use the PMTiles file: download or mirror it once and read it locally.

## Caching

| Header | Value |
|---|---|
| `Cache-Control` | `public, max-age=3600` (tiles, TileJSON) |
| `ETag` | version; with `If-None-Match` → `304` |

PMTiles files carry the release date in their name and never change; they may be cached indefinitely.

## Operations

- `GET https://api.mapx.at/health` – health check for monitoring.
- `GET https://api.mapx.at/v1/openapi.json` – OpenAPI description, static copy [openapi.yaml](/openapi.yaml).
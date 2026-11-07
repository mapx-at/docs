# Errors, limits and operations

RFC 9457 error format, request limits, caching, health check and OpenAPI.

## Error format

On errors, all endpoints respond with `application/problem+json` (RFC 9457), not HTML. Fields: `type` (`about:blank`), `title` (status text), `status`, `detail` (explanation).

| Status | Meaning |
|---|---|
| `400` | Parameters missing or malformed |
| `404` | Tileset or tile unknown, wrong extension, outside the zoom range |
| `429` | Too many requests, observe `Retry-After` |
| `503` | Service temporarily unavailable |

## Limits

The service limits requests per IP and responds above the limit with `429` and `Retry-After`.
On `429`, retry with increasing delay.

## Caching

Tiles and TileJSON carry `ETag` and `Cache-Control`; with `If-None-Match` the service responds `304`. PMTiles file names contain the reference date and never change, so they can be cached indefinitely.

## Health and OpenAPI

- `GET https://api.mapx.at/health`: liveness signal for monitoring.
- `GET https://api.mapx.at/v1/openapi.json`: description of all endpoints, static copy [openapi.yaml](/openapi.yaml).
# Errors, status codes and rate limits

Source: https://docs.mapx.at/api/errors/

> Error codes (problem+json), HTTP status codes, rate limits per zone and caching rules of all mapx APIs, with Retry-After on 429.

Every error is an `application/problem+json` document ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). The `type` is a URI into this page: `https://docs.mapx.at/api/errors/#<code>`. Branch on `code` (or `type`) and `status`, never on the text.

```json
{
  "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
  "title": "Invalid parameter",
  "status": 400,
  "detail": "types: unknown value 'foo' (allowed: address, street, locality, place, district, region, postcode, poi, road_ref)",
  "code": "invalid-parameter",
  "param": "types",
  "allowed": ["address", "street", "locality", "place", "district", "region", "postcode", "poi", "road_ref"]
}
```

## Fields

| Field | Meaning |
|---|---|
| `type` | URI of the error type (anchor on this page) |
| `title` | Short summary of the type |
| `status` | HTTP status |
| `detail` | Explanation for this request; English by default |
| `code` | Stable machine-readable code, equals the anchor |
| `param` | Offending parameter (where applicable); for path segments their name in the OpenAPI description (`coordinates`, `coordinate`, `category`), for fields of a JSON body the field path (`vehicles[0].start`) |
| `allowed` | Accepted values for enumerations (where applicable) |

## Authentication errors

Every request needs a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)). Authentication errors answer `401` or `403` as `application/problem+json`; the code is also in the response header `X-Mapx-Reason` (readable from browser scripts). Requests without a token get `401` `missing-token`.

These answers come from the token check in the gateway in front of the API. They have the same shape as all other errors: a `title` and `detail` per reason (for example `Access token required` for `missing-token`, `Invalid access token` for `invalid-token`), always English. Branch on `code` or `X-Mapx-Reason`.

```json
{"type": "https://docs.mapx.at/api/errors/#missing-token", "title": "Access token required", "status": 401, "detail": "Send the token as 'Authorization: Bearer <token>' or '?access_token=<token>'.", "code": "missing-token"}
```

The other errors the gateway produces itself (`404` `unknown-path`, `405`, `413`, `414`, `429`, `500`, `502`, `503`, `504`) are `application/problem+json` too, with `type`, `title`, `detail` and `code`. The gateway checks the method before the token: a method that is not allowed gives `405` even when the token is missing.

```bash
# with an invalid or unknown token (here the placeholder itself) the answer is 401 invalid-token
curl -si -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk" | head -12
```

## Language

`detail` is always English and names the exact limit or value (for example `limit: 1 to 10`). With `Accept-Language: de` the `title` is German, `detail` stays the same, so nothing gets lost; authentication errors and the other gateway errors (`405`, `413`, `414`, `429`, `5xx`) are always English (`unknown-path` is available in German). `type`, `code`, `param` and `allowed` never depend on the language. Responses carry `Content-Language` and `Vary: Accept-Language`.

## Catalog

| Code | Status | Title |
|---|---|---|
| [`bad-request`](#bad-request) | 400 | Bad request |
| [`missing-parameter`](#missing-parameter) | 400 | Missing parameter |
| [`invalid-parameter`](#invalid-parameter) | 400 | Invalid parameter |
| [`invalid-coordinates`](#invalid-coordinates) | 400 | Invalid coordinates |
| [`invalid-body`](#invalid-body) | 400 | Invalid request body |
| [`missing-token`](#missing-token) | 401 | Access token required |
| [`invalid-token`](#invalid-token) | 401 | Invalid access token |
| [`token-expired`](#token-expired) | 401 | Access token expired |
| [`token-blocked`](#token-blocked) | 403 | Access token blocked |
| [`origin-not-allowed`](#origin-not-allowed) | 403 | Origin not allowed |
| [`origin-missing`](#origin-missing) | 403 | Origin missing |
| [`pk-not-allowed`](#pk-not-allowed) | 403 | Public token not allowed here |
| [`app-not-allowed`](#app-not-allowed) | 403 | App not allowed |
| [`ip-not-allowed`](#ip-not-allowed) | 403 | IP not allowed |
| [`mcp-token-only`](#mcp-token-only) | 403 | MCP token only valid at the MCP server |
| [`mcp-token-required`](#mcp-token-required) | 403 | MCP token required |
| [`invalid-path`](#invalid-path) | 403 | Invalid request path |
| [`forbidden`](#forbidden) | 403 | Forbidden |
| [`range-required`](#range-required) | 403 | Range required |
| [`unknown-path`](#unknown-path) | 404 | Unknown path |
| [`not-found`](#not-found) | 404 | Not found |
| [`unknown-style`](#unknown-style) | 404 | Unknown style |
| [`unknown-tileset`](#unknown-tileset) | 404 | Unknown tileset |
| [`unknown-font`](#unknown-font) | 404 | Unknown font |
| [`unknown-parameter`](#unknown-parameter) | 400 | Unknown parameter |
| [`unknown-layer`](#unknown-layer) | 404 | Unknown layer |
| [`unknown-road`](#unknown-road) | 404 | Unknown road |
| [`km-out-of-range`](#km-out-of-range) | 404 | Kilometer out of range |
| [`unknown-id`](#unknown-id) | 404 | Unknown id |
| [`no-road-nearby`](#no-road-nearby) | 404 | No road nearby |
| [`tile-out-of-range`](#tile-out-of-range) | 404 | Tile out of range |
| [`outside-coverage`](#outside-coverage) | 404 | Outside coverage |
| [`method-not-allowed`](#method-not-allowed) | 405 | Method not allowed |
| [`payload-too-large`](#payload-too-large) | 413 | Payload too large |
| [`uri-too-long`](#uri-too-long) | 414 | URI too long |
| [`unprocessable`](#unprocessable) | 422 | Unprocessable request |
| [`rate-limited`](#rate-limited) | 429 | Rate limit exceeded |
| [`quota-exceeded`](#quota-exceeded) | 429 | Quota exceeded |
| [`internal-error`](#internal-error) | 500 | Internal error |
| [`upstream-error`](#upstream-error) | 502 | Upstream error |
| [`service-unavailable`](#service-unavailable) | 503 | Service unavailable |
| [`service-busy`](#service-busy) | 503 | Service busy |
| [`cache-unavailable`](#cache-unavailable) | 503 | Authorization backend unavailable |
| [`upstream-timeout`](#upstream-timeout) | 504 | Upstream timeout |

Titles of authentication errors and the other gateway errors are given as sent (English).

### bad-request

**400** · Bad request

The request is invalid in a way no more specific code covers.

What to do: Read `detail` and fix the request.

### missing-parameter

**400** · Missing parameter

A required parameter is missing. `param` names it. Where one of two parameters is required (`bbox` or `near` on `/v1/stops`, `bbox` or `corridor` on `/v1/speedlimits`) and both are missing, `param` names the first; if both are set, the answer is `invalid-parameter` with the second.

What to do: Add the parameter named in `param`.

### invalid-parameter

**400** · Invalid parameter

A parameter has a value that is malformed, out of range or not one of the accepted values. `param` names it; for enumerations `allowed` lists the accepted values.

What to do: Use a value from `allowed` or inside the range stated in `detail`.

### invalid-coordinates

**400** · Invalid coordinates

A coordinate (path segment or parameter) cannot be parsed or is outside the valid range. Unparsable path coordinates answer 400, not 404.

What to do: Send `lon,lat` in WGS84 degrees (longitude -180 to 180, latitude -90 to 90).

### invalid-body

**400** · Invalid request body

The body of a POST request is not valid JSON or not in the expected shape.

What to do: Check the JSON against the schema in the OpenAPI description.

### missing-token

**401** · Access token required

The request carries no token.

What to do: Create a token at [account.mapx.at](https://account.mapx.at/) and send it as `Authorization: Bearer YOUR_TOKEN` or `?access_token=YOUR_TOKEN`. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md).

### invalid-token

**401** · Invalid access token

The token is malformed or unknown (also after it was deleted or rotated).

What to do: Copy the token again from your account, check for a stray space or line break, and use the current token. Tokens start with `mx_pk_`, `mx_sk_` or (MCP server only) `mx_mcp_`.

### token-expired

**401** · Access token expired

The token is past its expiry date.

What to do: Create or rotate a token in your account.

### token-blocked

**403** · Access token blocked

The token was blocked in your account.

What to do: Unblock it in your account or use another token.

### origin-not-allowed

**403** · Origin not allowed

A public token (`mx_pk_`) was used from a website whose `Origin` (or `Referer` host) is not in the allowed origins of the token.

What to do: Add the origin to the token (for development also `localhost`), or call the API from your server with a secret token.

### origin-missing

**403** · Origin missing

A public token restricted to websites was used without `Origin` and without `Referer`, so the site cannot be checked.

What to do: Call from a browser page, or use a secret token on a server.

### pk-not-allowed

**403** · Public token not allowed here

A public token only allows read access (`GET`, `HEAD`) to the public products. Batch geocoding (`POST /v1/geocode/batch`), optimization (`POST /v1/optimization`), address validation via `POST /v1/address/validate`, traffic incidents via `POST /v1/traffic/incidents` and `GET /v1/corridor-graph` need a secret token.

What to do: Use a secret token (`mx_sk_`) from a server for this request.

### app-not-allowed

**403** · App not allowed

A public token bound to an Android app was used with a package name or signing certificate that does not match.

What to do: Send the headers `X-Mapx-Android-Package` and `X-Mapx-Android-Cert` of the registered app, or fix the binding in your account.

### ip-not-allowed

**403** · IP not allowed

The secret token is restricted to IP ranges and the request comes from another address.

What to do: Add the address (CIDR) to the token or send from an allowed host.

### mcp-token-only

**403** · MCP token only valid at the MCP server

A token of type AI assistant (`mx_mcp_`) was used for a direct API call. It only works at the [MCP server](https://docs.mapx.at/guides/mcp.md).

What to do: Use a secret token (`mx_sk_`) for direct API calls.

### mcp-token-required

**403** · MCP token required

The MCP server `https://mcp.mapx.at/mcp` only accepts tokens of type AI assistant (`mx_mcp_`); a public or secret token was sent.

What to do: Create a token of type AI assistant (MCP) at [account.mapx.at](https://account.mapx.at/tokens/new?kind=mcp).

### invalid-path

**403** · Invalid request path

The request path has an invalid percent-encoding, a NUL byte or leaves the root with `..`.

What to do: Send a normal path as listed in the [OpenAPI description](https://api.mapx.at/v1/openapi.json).

### forbidden

**403** · Forbidden

Access is refused without a more specific reason, for example a path that is blocked at the gateway.

What to do: Compare path and host with the [OpenAPI description](https://api.mapx.at/v1/openapi.json).

### range-required

**403** · Range required

`tiles.mapx.at` serves PMTiles files only in pieces: a `GET` on a `.pmtiles` file needs exactly one closed `Range` header (`bytes=<start>-<end>`). Requests without `Range`, with an open (`bytes=100-`), suffix (`bytes=-500`) or multiple range are refused. MapLibre with pmtiles.js and the mapx styles send valid ranges on their own. The gateway checks this before the token, like the method.

What to do: Load maps through the [style](https://docs.mapx.at/api/styles.md), [TileJSON or `{z}/{x}/{y}` tiles](https://docs.mapx.at/api/tiles.md). Whole files are not offered for download.

### unknown-path

**404** · Unknown path

The path does not exist.

What to do: Check the path against the [OpenAPI description](https://api.mapx.at/v1/openapi.json). An unknown routing profile such as the former `driving-ev` answers `400` `invalid-parameter`.

### not-found

**404** · Not found

The addressed resource (a tile, a sprite, ...) does not exist.

What to do: Check the path. Tiles inside the zoom range without data answer `204`, not this error.

### unknown-style

**404** · Unknown style

The style id is not known.

What to do: List styles with `/v1/styles`.

### unknown-tileset

**404** · Unknown tileset

The tileset name is not known.

What to do: List tilesets with `/v1/tiles`.

### unknown-font

**404** · Unknown font

The font name is not known.

What to do: List fonts with `/v1/fonts`.

### unknown-parameter

**400** · Unknown parameter

The endpoint does not know a query parameter or a field of the JSON body (often a typo, e.g. `limt` instead of `limit`). `param` names it, `detail` lists the known parameters of the endpoint. `access_token` is accepted everywhere.

What to do: Fix the name or drop the parameter; the parameters of each endpoint are in the [OpenAPI description](https://api.mapx.at/v1/openapi.json).

### unknown-layer

**404** · Unknown layer

A weather or air-quality layer (path of `/v1/weather/tiles/…`, `/v1/air-quality/tiles/…`) is not known.

What to do: Use one of temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast, pt_nowcast (list: `/v1/weather/tiles/<param>.json`).

### unknown-road

**404** · Unknown road

`/v1/road-km` has no kilometre points for this road number (`param` = `road`), or none in the requested state (`state`). Road numbers are A, S, B or L plus up to four digits (`B1`, `L100`, `L1234a`).

What to do: Check the number; `/v1/road-km/roads` lists the roads with kilometre points.

### km-out-of-range

**404** · Kilometer out of range

The road is known but has no point at this kilometre (beyond its end or inside a gap such as a kilometre jump). The problem object has the extra field `ranges`: `[{"state": "Niederösterreich", "from": 0.0, "to": 174.6}]`, the contiguous ranges in km.

What to do: Pick a kilometre inside one of the `ranges`.

### unknown-id

**404** · Unknown id

A suggestion, address or place id is not known (or expired after a data update).

What to do: Search again and use the id from the fresh response. For `/v1/departures` with an unknown station id `param` names `stop`.

### no-road-nearby

**404** · No road nearby

No road usable by the profile lies within the search radius around a point (routing, isochrone, matrix: 1000 m by default, per point via `radiuses`), or within the buffer along the line of `/v1/speedlimits` (`corridor`) and `/v1/corridor-graph` (`polyline`). `param` names the parameter (`coordinates`, `coordinate`, `corridor`, `polyline`), `detail` the point (`start`, `destination`) and the radius.

What to do: Move the point closer to a road or enlarge the radius (`radiuses`, `buffer`); check the coordinate order `lon,lat`.

### tile-out-of-range

**404** · Tile out of range

The tile is outside the zoom or tile range of the tileset or style.

What to do: Stay within `minzoom`/`maxzoom` and `bounds` of the TileJSON.

### outside-coverage

**404** · Outside coverage

The point lies outside the area for which the data exist (weather, nowcast, air quality and climate cover Austria and its surroundings). `detail` names the data and adds "(lon and lat swapped?)" when the swapped coordinates would be inside.

What to do: Send a point in Austria as `lon,lat` (longitude about 9.5 to 17.2, latitude about 46.4 to 49.0).

### method-not-allowed

**405** · Method not allowed

Only GET (and HEAD) is allowed, POST only for batch geocoding, optimization, address validation and traffic incidents. The `Allow` header lists the allowed methods (e.g. `POST /v1/geocode` → `405`, `Allow: GET, HEAD`). The gateway checks the method before the token, so `405` also comes when the token is missing as well.

What to do: Use a method from `Allow`.

### payload-too-large

**413** · Payload too large

The request body is larger than the endpoint accepts: batch geocoding 512 KB, optimization 256 KB, traffic incidents 64 KB, all other POST endpoints (such as address validation) 16 KB; the MCP server 256 KB.

What to do: Split or shrink the request.

### uri-too-long

**414** · URI too long

The request URL is longer than about 8 KB, for example very many `points` on `/v1/elevation`.

What to do: Send fewer values per request or, where the endpoint offers it, use POST with a JSON body.

### unprocessable

**422** · Unprocessable request

The request is well-formed but cannot be answered, e.g. no route between the points, no map match, no order for an optimization, no stop within reach or a date outside the timetable period.

What to do: Change the points or the time; read `detail`.

### rate-limited

**429** · Rate limit exceeded

Too many requests. `429` has two causes: the limit per IP address and zone (endpoint group) or the per-minute limit of the token. Both answer with `title` `Rate limit exceeded`, a `detail` and `Retry-After`. See [rate limits](https://docs.mapx.at/api/errors.md#rate-limits).

Expensive groups also have an overall limit for all clients together; reaching it answers the same way.

What to do: Wait `Retry-After` seconds, then retry with growing back-off. The `RateLimit-Policy` header of every response (set by the gateway from its zone table) shows the zone.

The usage view of your account lists the per-minute limit of a token as `rate-limited` and the limit of a zone as `rate-limited:<zone>` (e.g. `rate-limited:search`, or `rate-limited:connections` for too many simultaneous connections), each under the token of the request.

### quota-exceeded

**429** · Quota exceeded

The daily or monthly limit of the account is used up (all tokens of the account together, days and months in UTC). Currently both limits are unlimited, see [access and limits](https://docs.mapx.at/start/access-and-limits.md#rate-limits).

What to do: `Retry-After` gives the seconds until the next day or month (UTC); retrying earlier does not help. Your account shows the usage under Usage.

### internal-error

**500** · Internal error

An unexpected error occurred on the server or in the gateway.

What to do: Retry once; if it persists, report the request.

### upstream-error

**502** · Upstream error

A backing service (search, router) did not answer or answered with an error.

What to do: Retry after a few seconds.

### service-unavailable

**503** · Service unavailable

The data or service is temporarily unavailable (e.g. weather data not loaded yet, a style being rebuilt).

What to do: Retry later.

### service-busy

**503** · Service busy

The server is at its concurrency limit.

What to do: Wait `Retry-After` seconds and retry.

### cache-unavailable

**503** · Authorization backend unavailable

The store behind the token check is briefly unreachable and the token is not in the local cache. The request is refused to stay on the safe side.

What to do: Retry after a few seconds.

### upstream-timeout

**504** · Upstream timeout

The service behind the gateway did not answer in time.

What to do: Retry after a few seconds; split large requests.

See also: [Access and limits](https://docs.mapx.at/start/access-and-limits.md) for rate limits and caching.

## Rate limits

Limits per IP address and zone and the per-minute limit per token are listed under [Access and limits](https://docs.mapx.at/start/access-and-limits.md). Retry on `429` and `503` with growing delay and honor `Retry-After`; do not retry `400`, `401`, `403`, `404` and `422` unchanged.

## Caching

| Header | Value |
|---|---|
| `Cache-Control` | `private` for every response to a request with a token (browser cache only, no shared caches), with `max-age=3600` (tiles.mapx.at, TileJSON, elevation), `max-age=86400` (tiles via api.mapx.at), `max-age=60` (Search Box suggestions, traffic), `max-age=30` (departures), `max-age=300` (geocoding, reverse, autofill suggestions, address validation via GET, tileset list), `max-age=0` (routing, matrix, isochrone, POST requests); `/health` `no-store` |
| `ETag` | on every `GET` response; with `If-None-Match` → `304` without body. Tiles, styles, TileJSON and OpenAPI: version of the file (plus `Last-Modified`, `If-Modified-Since`), all others: hash of the content |
| `Server-Timing` | `router`, `search`, `render`, `total` in milliseconds |
| `RateLimit-Policy` | zone and limit of the endpoint (set by the gateway) |

PMTiles files carry the release date in their name and never change; they may be cached indefinitely.

## Attribution

JSON responses carrying data have a top-level `attribution` field (plain text, no HTML) naming the source to credit when displaying or passing on the data. It always starts with `mapx.at | `, in the form `mapx.at | <source> (<license>); …` without prefix words, e.g. `mapx.at | GeoSphere Austria (CC BY 4.0)` for weather or `mapx.at | GIP.at (CC BY 4.0)` for routing, matrix and isochrone. The text is the same for every `Accept-Language`. Exceptions: the lists `/v1/styles`, `/v1/fonts`, `/v1/sprites` (arrays); styles credit their sources in `sources`, `mapx.at` as the first source `mapx`. See [Attribution](https://docs.mapx.at/reference/attribution.md).

# Errors and limits


Every error is an `application/problem+json` document ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). The `type` is a URI into this page: `https://docs.mapx.at/api/errors/#<code>`. Branch on `code` (or `type`) and `status`, never on the text.

```json
{
  "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
  "title": "Invalid parameter",
  "status": 400,
  "detail": "types: unknown value 'foo' (allowed: address, street, locality, place, district, region, postcode, poi)",
  "code": "invalid-parameter",
  "param": "types",
  "allowed": ["address", "street", "locality", "place", "district", "region", "postcode", "poi"]
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
| `param` | Offending parameter (where applicable) |
| `allowed` | Accepted values for enumerations (where applicable) |

## Authentication errors

Every request needs a token ([Access and limits](/start/access-and-limits/)). Authentication errors answer `401` or `403` as `application/problem+json`; the code is also in the response header `X-Mapx-Reason` (readable from browser scripts). Anonymous requests are still served during the transition period; they carry `X-Mapx-Token-Warning: missing-token` and will get `401` later.

```bash
# with an invalid or unknown token (here the placeholder itself) the answer is 401 invalid-token
curl -si -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk" | head -12
```

## Language

`title` and `detail` are English by default. Send `Accept-Language: de` to get German texts; `type`, `code`, `param` and `allowed` stay identical. Responses carry `Content-Language` and `Vary: Accept-Language`. German `detail` is the generic description of the code; the English `detail` is the most specific.

## Catalog

| Code | Status | Title |
|---|---|---|
| [`bad-request`](#bad-request) | 400 | Bad request |
| [`missing-parameter`](#missing-parameter) | 400 | Missing parameter |
| [`invalid-parameter`](#invalid-parameter) | 400 | Invalid parameter |
| [`invalid-coordinates`](#invalid-coordinates) | 400 | Invalid coordinates |
| [`invalid-body`](#invalid-body) | 400 | Invalid request body |
| [`missing-token`](#missing-token) | 401 | Token missing |
| [`invalid-token`](#invalid-token) | 401 | Invalid token |
| [`token-expired`](#token-expired) | 401 | Token expired |
| [`token-blocked`](#token-blocked) | 403 | Token blocked |
| [`origin-not-allowed`](#origin-not-allowed) | 403 | Origin not allowed |
| [`origin-missing`](#origin-missing) | 403 | Origin missing |
| [`pk-not-allowed`](#pk-not-allowed) | 403 | Public token not allowed here |
| [`app-not-allowed`](#app-not-allowed) | 403 | App not allowed |
| [`ip-not-allowed`](#ip-not-allowed) | 403 | IP address not allowed |
| [`unknown-path`](#unknown-path) | 404 | Unknown path |
| [`not-found`](#not-found) | 404 | Not found |
| [`unknown-style`](#unknown-style) | 404 | Unknown style |
| [`unknown-tileset`](#unknown-tileset) | 404 | Unknown tileset |
| [`unknown-font`](#unknown-font) | 404 | Unknown font |
| [`unknown-parameter`](#unknown-parameter) | 404 | Unknown parameter |
| [`unknown-id`](#unknown-id) | 404 | Unknown id |
| [`tile-out-of-range`](#tile-out-of-range) | 404 | Tile out of range |
| [`method-not-allowed`](#method-not-allowed) | 405 | Method not allowed |
| [`unprocessable`](#unprocessable) | 422 | Unprocessable request |
| [`rate-limited`](#rate-limited) | 429 | Rate limit exceeded |
| [`internal-error`](#internal-error) | 500 | Internal error |
| [`upstream-error`](#upstream-error) | 502 | Upstream error |
| [`service-unavailable`](#service-unavailable) | 503 | Service unavailable |
| [`service-busy`](#service-busy) | 503 | Service busy |

### bad-request

**400** · Bad request

The request is invalid in a way no more specific code covers.

What to do: Read `detail` and fix the request.

### missing-parameter

**400** · Missing parameter

A required parameter is missing. `param` names it.

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

**401** · Token missing

The request carries no token. Today anonymous requests are still answered during the transition period and carry the header `X-Mapx-Token-Warning`; afterwards they get this error.

What to do: Create a token at [account.mapx.at](https://account.mapx.at/) and send it as `Authorization: Bearer YOUR_TOKEN` or `?access_token=YOUR_TOKEN`. See [Access and limits](/start/access-and-limits/).

### invalid-token

**401** · Invalid token

The token is malformed or unknown (also after it was deleted or rotated).

What to do: Copy the token again from your account, check for a stray space or line break, and use the current token. Tokens start with `mx_pk_` or `mx_sk_`.

### token-expired

**401** · Token expired

The token is past its expiry date.

What to do: Create or rotate a token in your account.

### token-blocked

**403** · Token blocked

The token was blocked in your account.

What to do: Unblock it in your account or use another token.

### origin-not-allowed

**403** · Origin not allowed

A public token (`mx_pk_`) was used from a website whose `Origin` (or `Referer` host) is not in the allowed origins of the token.

What to do: Add the origin to the token (for development also `localhost`), or call the API from your server with a secret token.

### origin-missing

**403** · Origin missing

A public token was used without `Origin` and without `Referer`, so the site cannot be checked.

What to do: Call from a browser page, or use a secret token on a server.

### pk-not-allowed

**403** · Public token not allowed here

A public token only allows read access (`GET`, `HEAD`) to the public products.

What to do: Use a secret token (`mx_sk_`) from a server for this request.

### app-not-allowed

**403** · App not allowed

A public token bound to an Android app was used with a package name or signing certificate that does not match.

What to do: Send the headers `X-Mapx-Android-Package` and `X-Mapx-Android-Cert` of the registered app, or fix the binding in your account.

### ip-not-allowed

**403** · IP address not allowed

The secret token is restricted to IP ranges and the request comes from another address.

What to do: Add the address (CIDR) to the token or send from an allowed host.

### unknown-path

**404** · Unknown path

The path does not exist.

What to do: Check the path against the [OpenAPI description](https://api.mapx.at/v1/openapi.json). The former aliases `/v1/{tileset}...` and `driving-ev` were removed.

### not-found

**404** · Not found

The addressed resource (a tile, a sprite, ...) does not exist.

What to do: For tiles treat as an empty tile.

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

**404** · Unknown parameter

A weather parameter name is not known.

What to do: Use one of temperature, precipitation, wind, humidity, cloud, pressure.

### unknown-id

**404** · Unknown id

A suggestion, address or place id is not known (or expired after a data update).

What to do: Search again and use the id from the fresh response.

### tile-out-of-range

**404** · Tile out of range

The tile is outside the zoom or tile range of the tileset or style.

What to do: Stay within `minzoom`/`maxzoom` and `bounds` of the TileJSON.

### method-not-allowed

**405** · Method not allowed

Only GET (and HEAD) is allowed, POST only for batch geocoding, optimization, address validation and traffic incidents.

What to do: Use the documented method.

### unprocessable

**422** · Unprocessable request

The request is well-formed but cannot be answered, e.g. no route between the points or a date outside the timetable period.

What to do: Change the points or the time; read `detail`.

### rate-limited

**429** · Rate limit exceeded

Too many requests from your token (without a valid token: your IP address) in this endpoint group. See [rate limits](/api/errors/#rate-limits).

What to do: Wait `Retry-After` seconds, then retry with growing back-off. The `RateLimit-Policy` header of every response shows the zone.

### internal-error

**500** · Internal error

An unexpected error occurred on the server.

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

See also: [Errors and limits](/api/errors/) for rate limits and caching.

## Rate limits

Limits per token and zone are listed under [Access and limits](/start/access-and-limits/). Retry on `429` and `503` with growing delay and honor `Retry-After`; do not retry `400`, `401`, `403`, `404` and `422` unchanged.

## Caching

| Header | Value |
|---|---|
| `Cache-Control` | `public, max-age=3600` (tiles, TileJSON, elevation, parcel), `max-age=300` (geocoding, reverse, autofill suggestions, address validation via GET, tileset list), `max-age=0` (routing, matrix, isochrone, POST requests) |
| `ETag`, `Last-Modified` | version; with `If-None-Match` / `If-Modified-Since` → `304` (tiles, styles, TileJSON, OpenAPI) |
| `Server-Timing` | `router`, `search`, `render`, `total` in milliseconds |
| `RateLimit-Policy` | zone and limit of the endpoint |

PMTiles files carry the release date in their name and never change; they may be cached indefinitely.

## Attribution

JSON responses carrying data have a top-level `attribution` field (plain text, no HTML) naming the source to credit when displaying or passing on the data, e.g. `Datenquelle: BEV – DKM, CC BY 4.0` for parcels or `Datenquelle: GIP.at …` for routing, matrix and isochrone. Exceptions: the lists `/v1/styles`, `/v1/fonts`, `/v1/sprites` (arrays); styles credit their sources in `sources`. See [Attribution](/reference/attribution/).


# Error catalog

Every error code of the mapx API: type URI, status, meaning and what to do.

Every error is an `application/problem+json` document ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). The `type` is a URI into this page: `https://docs.mapx.at/api/errors/#<code>`. Branch on `code` (or `type`) and `status`, never on the text.

<Code lang="json" code={`{
  "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
  "title": "Invalid parameter",
  "status": 400,
  "detail": "types: unknown value 'foo' (allowed: address, street, locality, place, district, region, postcode, poi)",
  "code": "invalid-parameter",
  "param": "types",
  "allowed": ["address", "street", "locality", "place", "district", "region", "postcode", "poi"]
}`} />

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

Too many requests from your IP address in this endpoint group. See [rate limits](/en/api/fehler/#rate-limits).

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

See also: [Errors and limits](/en/api/fehler/) for rate limits and caching.
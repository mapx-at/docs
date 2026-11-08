# Access and limits

## Get a token

All endpoints under `https://api.mapx.at/v1/`, `https://routing.mapx.at/`, `https://search.mapx.at/` and all tiles under `https://tiles.mapx.at/` need a token. It is free and there is no usage-based charge.

1. Open [account.mapx.at](https://account.mapx.at/) and register with your e-mail address (an 8-digit code is sent to you) or sign in with Google or Microsoft. You must agree to be contacted about the service and your account (security, changes, shutdown). Marketing mail is a separate, optional checkbox.
2. Under **Tokens** create as many named tokens as you like. The token is shown once; copy it then. You can block, rotate or delete a token at any time, deleting takes effect immediately.
3. Send the token with every request (see below). The **Usage** page shows requests per token, product and day, including rejected requests with the reason.

The OpenAPI description `https://api.mapx.at/v1/openapi.json` and CORS preflight requests (`OPTIONS`) work without a token.

## Two kinds of token

| Kind | Prefix | Use | Restrictions |
|---|---|---|---|
| Public | `mx_pk_…` | Websites and apps. The token is visible in the page source or in the APK, so it only works where you allow it. | Read-only products, `GET`/`HEAD` only. Bound to **allowed origins** (for example `https://example.com`, `*.example.com`) or to an **Android app** (package name plus SHA-256 of the signing certificate). |
| Secret | `mx_sk_…` | Servers, scripts and CI. Never put it into a web page or an app. | No origin binding; optionally bound to IP ranges (CIDR). |

A public token is checked against the `Origin` header (for tiles and images without one, against the `Referer` host). Requests from other sites get `403` with the reason `origin-not-allowed`. Add `localhost` to the origins only while you develop.

## Send the token

Use the header in server code and scripts. In the browser use the query parameter: images and MapLibre tile URLs cannot set headers, and the CORS preflight does not allow the `Authorization` header.

```bash
# header
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk&limit=1" | jq

# query parameter
curl -s "https://api.mapx.at/v1/geocode?q=Melk&limit=1&access_token=YOUR_TOKEN" | jq
```

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Melk&access_token=YOUR_TOKEN"); // in the browser use access_token; Node can use the header
```

**Maps with MapLibre GL JS.** The simplest way is a `transformRequest` that adds the token to every request to `mapx.at`, never to other hosts:

```js
const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets",
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
});
```

When you request a style with `?access_token=…`, the API writes the token into all `mapx.at` URLs of the style (tiles, glyphs, sprites). Clients that cannot set `transformRequest` then work with `style: "https://api.mapx.at/v1/styles/streets?access_token=YOUR_TOKEN"`. Such a response is `private` and must not be shared.

**With the SDK.** `mapx.map({ accessToken: "YOUR_TOKEN", … })` or `mapx.setAccessToken("YOUR_TOKEN")` once; all helpers, styles, tiles and `staticImageUrl()` then carry the token. See [SDK](/guides/sdk/).

**PMTiles files.** Append the token to the file URL: `pmtiles://https://tiles.mapx.at/<file>.pmtiles?access_token=YOUR_TOKEN`.

**Android.** Send `Authorization: Bearer <public token>` plus `X-Mapx-Android-Package` (package name) and `X-Mapx-Android-Cert` (SHA-256 of the signing certificate, read at run time with `PackageManager`).

Without a valid token the API answers `401` (`missing-token`, `invalid-token`, `token-expired`) or `403` (`token-blocked`, `pk-not-allowed`, `origin-missing`, `origin-not-allowed`, `app-not-allowed`, `ip-not-allowed`) as `application/problem+json`; see [Errors and limits](/api/errors/). During the transition period the token may still be optional: requests without one succeed and carry the header `X-Mapx-Token-Warning`.

## Rate limits

Limits apply per token and zone (requests without a valid token count per IP address). Every response carries a `RateLimit-Policy` header, for example `"search";q=10;w=1;burst=30` (10 requests per second plus a burst of 30). Exceeding a limit returns `429` with `Retry-After`. On top of the zones a token may make 6000 requests per minute by default; rejected requests with `rate-limited` are counted.

| Zone | Requests | Used by |
|---|---|---|
| `tiles` | 100/s, burst 200 | tiles, TileJSON, glyphs, sprites |
| `search` | 10/s, burst 30 | geocoding, Search Box, autofill, tilequery, parcel, departures, weather, traffic |
| `route` | 5/s, burst 20 | directions, transit journeys |
| `elevation` | 10/s, burst 20 | elevation and profile |
| `static` | 5/s, burst 60 | Static Tiles, traffic tiles |
| `heavy` | 30/min, burst 5 | static images, isochrone, matrix, matching, optimization, batch geocoding |

For bulk work, download the [PMTiles](/tilesets/) file once instead of requesting millions of tiles. Details and retry advice: [Errors and limits](/api/errors/).

## CORS and caching

CORS allows every origin, so you can call the APIs straight from the browser. The preflight answer of the API does not list the `Authorization` header yet, so in browser code send the token as `access_token` in the URL. Responses without a token in the URL are cacheable by shared caches; responses that contain your token (styles requested with `access_token`) are `private`.

## Attribution

Show the source of the data wherever the map or result is visible. MapLibre adds the text for you when you load a style or TileJSON. See [Attribution](/reference/attribution/).

## Next

- [First map in 5 minutes](/start/first-map/)
- [Errors and limits](/api/errors/)


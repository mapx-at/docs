# API token, access and rate limits

Source: https://docs.mapx.at/start/access-and-limits/

> Get a free mapx API token, learn the difference between public and secret tokens, how to send them, rate limits per zone, CORS and caching.

## Get a token

All endpoints under `https://api.mapx.at/v1/` and all tiles under `https://tiles.mapx.at/` need a token. It is free.

1. Open [account.mapx.at](https://account.mapx.at/) and sign in with your e-mail address (an 8-digit code is sent to you) or with Google or Microsoft. If there is no account for the address yet, you create it after entering the code with “Create account”; you tick the box for the terms of use, the privacy policy and contact about the service and your account (security, changes, shutdown). The newsletter is optional, there or later in your account. Code emails are limited per address and per connection; if you see “Too many attempts”, try again a little later.
2. Under **Tokens** use **New token** to create a named token (up to 100 per account). The token is shown once; copy it then.
3. Send the token with every request (see below).

### Managing tokens and your account

- **Edit:** You can change the name, allowed origins or Android app, IP ranges and expiry date later; changes take effect immediately. Wildcards need your own domain after them (`*.example.com`, not `*.at`), and an IP range `/0` is not allowed (leave the field empty instead).
- **Expiry date:** given in UTC. Once set, an expiry date can only be brought forward, not extended.
- **Rotate:** creates a new token with the same settings; the old one keeps working for 7 days so you can replace it calmly.
- **Block:** the token is rejected immediately (`token-blocked`) and can be unblocked again. **Delete** takes effect immediately and permanently.
- **Usage:** the **Overview** and **Usage** pages show requests per token, product and day, rejected ones with the reason (e.g. `origin-not-allowed`). “Last used” is updated about once a minute.
- **Account:** under **Account** you can change your address (shortly after signing in; a code is sent to the new address), turn the newsletter on or off, export all your data as JSON and delete the account. Deleting the account deletes all tokens; applications using them stop working immediately.

The OpenAPI description `https://api.mapx.at/v1/openapi.json` and CORS preflight requests (`OPTIONS`) work without a token.

## Kinds of token

| Kind | Prefix | Use | Restrictions |
|---|---|---|---|
| Public | `mx_pk_…` | Websites and apps. The token is visible in the page source or in the APK, so it only works where you allow it. | Read-only products, `GET`/`HEAD` only. Not for `POST /v1/optimization`, `POST /v1/geocode/batch` and `/v1/corridor-graph` (otherwise `403 pk-not-allowed`). Bound to **allowed origins** (for example `https://example.com`, `*.example.com`) or to an **Android app** (package name plus SHA-256 of the signing certificate). |
| Secret | `mx_sk_…` | Servers, scripts and CI. Never put it into a web page or an app. | No origin binding; optionally bound to IP ranges (CIDR). |
| AI assistant (MCP) | `mx_mcp_…` | AI assistants and agents that use the [mapx MCP server](https://docs.mapx.at/guides/mcp.md). | Created by hand or by signing in with your account (then shown as "connected via &lt;host&gt;"). Works only at `https://mcp.mapx.at/mcp`, no origin or IP binding, optional expiry date. Direct API calls with it answer `403` `mcp-token-only`. |

A public token is checked against the `Origin` header (for tiles and images without one, against the `Referer` host). Requests from other sites get `403` with the reason `origin-not-allowed`. An origin is scheme, host and port, for example `http://localhost:3000` while you develop; add `localhost` only during development. A page opened as a file (`file://`) has no origin: use a local web server (`npx serve`).

## Send the token

Use the header in server code and scripts. In the browser `fetch` calls can send the header as well; use the query parameter where no header can be set, for example in image URLs and MapLibre tile URLs.

```bash
# header
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk&limit=1" | jq

# query parameter
curl -s "https://api.mapx.at/v1/geocode?q=Melk&limit=1&access_token=YOUR_TOKEN" | jq
```

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Melk&access_token=YOUR_TOKEN"); // the query parameter works everywhere; the header works in the browser too
```

**Maps with MapLibre GL JS.** The simplest way is a `transformRequest` that adds the token to every request to `mapx.at`, never to other hosts:

```js
const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
// the mapx styles load their tiles from PMTiles files: register the protocol once
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets",
  center: [16.3725, 48.2085], zoom: 13, // Vienna, lon,lat
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```

**PMTiles prerequisite.** The mapx styles load their tiles from PMTiles files. The page must therefore include `pmtiles.js` (`<script src="https://unpkg.com/pmtiles@4.5.0/dist/pmtiles.js"></script>`) and call `maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile)` once; otherwise the map stays empty (console: `URL scheme "pmtiles" is not supported`). Your own styles that only use TileJSON sources (`https://tiles.mapx.at/v1/tiles/{tileset}.json`) do not need it.

When you request a style with `?access_token=…`, the API writes the token into all `mapx.at` URLs of the style (tiles, glyphs, sprites, PMTiles files). That is why the `transformRequest` above adds the token only when `access_token` is not in the URL yet. Clients that cannot set `transformRequest` work with `style: "https://api.mapx.at/v1/styles/streets?access_token=YOUR_TOKEN"` (they still need the PMTiles protocol). Such a response is `private` and must not be shared.

**Android.** Send `Authorization: Bearer <public token>` plus `X-Mapx-Android-Package` (package name) and `X-Mapx-Android-Cert` (SHA-256 of the signing certificate, read at run time with `PackageManager`).

Without a valid token the API answers `401` (`missing-token`, `invalid-token`, `token-expired`) or `403` (`token-blocked`, `pk-not-allowed`, `origin-missing`, `origin-not-allowed`, `app-not-allowed`, `ip-not-allowed`) as `application/problem+json`; see [Errors and limits](https://docs.mapx.at/api/errors.md).

## Rate limits

The zones below apply per IP address (per token over MCP). On top of that, each token may make 6000 requests per minute by default; rejected requests are counted too and show up in the usage view under the token of the request, as `rate-limited` (per-minute limit) or `rate-limited:<zone>` (zone limit, e.g. `rate-limited:search`). Every API response carries a `RateLimit-Policy` header, for example `"search";q=10;w=1;burst=30` (10 requests per second plus a burst of 30); the gateway sets it from the same table as its limits, so it always names the zone that actually applies. The PMTiles requests of the styles (`https://tiles.mapx.at/…pmtiles`) do not carry it; they count in the `tiles` zone. Exceeding a limit returns `429` with `Retry-After`. A daily and a monthly limit per account are also planned (all tokens together, UTC; when exceeded: `429` `quota-exceeded` with `Retry-After` until the next day or month); currently both are unlimited. Your account shows the current state under Usage.

| Zone | Requests | Used by |
|---|---|---|
| `tiles` | 100/s, burst 200 | tiles, TileJSON, glyphs and sprites (burst 100), PMTiles requests of the styles (burst 500 on `tiles.mapx.at`) |
| `search` | 10/s, burst 30 | styles and the lists `/v1/styles`, `/v1/tiles`, geocoding, Search Box, autofill, address validation, tilequery, departures, stops, weather values, traffic reports |
| `route` | 5/s, burst 20 | directions, transit journeys, speed limits (corridor graph: burst 10) |
| `elevation` | 10/s, burst 20 | elevation and profile |
| `static` | 5/s, burst 60 | Static Tiles, traffic tiles |
| `weather` | 20/s, burst 200 | weather tiles (a radar animation loads 70 to 150 tiles at once) |
| `heavy` | 30/min, burst 5 | static images, isochrone, matrix, matching, optimized trips, optimization (burst 3) |
| `batch` | 2/min, burst 2 | batch geocoding (up to 1000 addresses per request, one request at a time) |

Expensive groups (rendered tiles and static images, routing, search, batch geocoding) also have an overall limit for all clients together, so that a single client cannot use up the server. It answers `429` with `Retry-After` as well.

**Over MCP.** Every tool call of the [MCP server](https://docs.mapx.at/guides/mcp.md) counts like an API call (some tools make several) and shows up as MCP in the usage view. The per-minute limit of the token and the zone limits above apply, per token instead of per IP address. The MCP endpoint itself accepts 10 requests per second per IP address (burst 40) and request bodies up to 256 KB.

For many tiles: cache responses in the client (tiles change only with a new release) and on `429` retry after `Retry-After`. Details and retry advice: [Errors and limits](https://docs.mapx.at/api/errors.md).

## CORS and caching

CORS allows every origin, so you can call the APIs straight from the browser. The preflight answer allows the `Authorization` header, so `fetch` calls from the browser can send the token in the header; image and tile URLs take it as `access_token`. If a request carries both a `Bearer` token and `access_token`, the `Bearer` token counts; another `Authorization` scheme (for example `Basic` of a proxy) does not hide `access_token`.

All responses to requests with a token are `Cache-Control: private`: the browser may cache them, shared caches (proxies, CDNs) must not, so no one else gets your response without a token. Every `GET` response carries an `ETag`; send it back as `If-None-Match` and you get `304 Not Modified` without a body (tiles, styles and TileJSON use the file version, all other responses the content, for example traffic reports polled every minute).

## Attribution

Show the source of the data wherever the map or result is visible. MapLibre adds the text for you when you load a style or TileJSON. See [Attribution](https://docs.mapx.at/reference/attribution.md).

## No warranty

All mapx data and results are provided without guarantee and used at your own risk. Check results yourself, especially for addresses, routes, departures, weather. The [terms of use](https://mapx.at/en/terms/) apply.

## Next

- [First map in 5 minutes](https://docs.mapx.at/start/first-map.md)
- [Errors and limits](https://docs.mapx.at/api/errors.md)

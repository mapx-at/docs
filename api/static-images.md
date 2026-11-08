# Static Images API

<!-- Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. -->

Renders a map image for a view and size, optionally with overlays. The result is cacheable (`ETag`).

Base URL: `https://api.mapx.at` · Rate-limit zone: `heavy` (30/min) · Guide: [Static images](/guides/static-images/)

## Map image (Static Images, Mapbox form)

`GET /v1/static/{style}/{view}/{size}`

`satellite` (BEV orthophoto, z5 to z18) or `outdoors` (hypsometric tints with hillshade from the DTM, z5 to z16). The `streets*` styles are rendered server-side from the style and the vector tiles (fills, lines, relief shading, place, road and water names; icons from the sprite, house numbers from z16; z0 to z22), usually under 0.2 s per 600x400.

<details>
<summary>More</summary>

`satellite-streets` draws the orthophoto (z5 to z18) with roads, boundaries, contours, names and icons of the vector map on top. Output PNG (default) or WebP, at most 1280x1280 (2560x2560 pixels with @2x). `Cache-Control` and `ETag` (If-None-Match gives 304).

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `style` | path | string | yes | – | streets, streets-dawn, streets-dusk, streets-night, satellite, satellite-streets or outdoors (old ids `karte`, `luftbild`, ... as deprecated alias) |
| `view` | path | string | yes | – | lon,lat,zoom[,bearing] (rotation in degrees, pitch only 0) or `auto` (only with overlays, fits the extent) |
| `size` | path | string | yes | – | \{width\}x\{height\}[@2x][.png\|.webp], each 1 to 1280 |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/static/satellite/16.3725,48.2085,15/600x400" -o result.png
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/static/satellite/16.3725,48.2085,15/600x400", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/static/satellite/16.3725,48.2085,15/600x400", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.png", "wb").write(res.content)
```


**Example response**: `image/png`, `image/webp` · Image

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `304` | Not modified (with `If-None-Match`) |
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Map image with overlays

`GET /v1/static/{style}/{overlay}/{view}/{size}`

`overlay`: comma-separated list (at most 50, 20000 vertices) of `pin-{s|m|l}[-label]+{hex}(lon,lat)` (label not drawn), `path-{width}[+{stroke}[-{opacity}][+{fill}[-{opacity}]]]({polyline, precision 5, URL-encoded})` and `geojson({URL-encoded GeoJSON})` with simplestyle properties (stroke, stroke-width, stroke-opacity, fill, fill-opacity, marker-color, marker-size).

<details>
<summary>More</summary>

Example: `/v1/static/satellite/pin-s+ff0000(15.33,48.22)/15.33,48.22,15/600x400.png`

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `style` | path | string | yes | – | streets, streets-dawn, streets-dusk, streets-night, satellite, satellite-streets or outdoors (old ids `karte`, `luftbild`, ... as deprecated alias) |
| `overlay` | path | string | yes | – | pin-..., path-..., geojson(...), comma-separated |
| `view` | path | string | yes | – | lon,lat,zoom[,bearing] (rotation in degrees, pitch only 0) or `auto` (only with overlays, fits the extent) |
| `size` | path | string | yes | – | \{width\}x\{height\}[@2x][.png\|.webp], each 1 to 1280 |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/static/streets/pin-s%2Bff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400" -o result.png
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/static/streets/pin-s%2Bff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/static/streets/pin-s%2Bff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.png", "wb").write(res.content)
```


**Example response**: `image/png`, `image/webp` · Image

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `304` | Not modified (with `If-None-Match`) |
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Playground




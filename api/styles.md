# Styles API

Ready-made MapLibre styles, glyphs, sprites and rendered raster tiles.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Ready-made styles for MapLibre GL JS and compatible clients, plus the fonts and sprites they use. Static Tiles render a style into raster tiles for clients without vector support.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s), `tiles` (100/s), `static` (5/s) · Guide: [Maps](/guides/maps/)

## List styles

`GET /v1/styles`

Available style ids: streets, streets-dawn, streets-dusk, streets-night, traffic-day, satellite, satellite-streets, outdoors.

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/styles" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
[
  {
    "id": "streets",
    "url": "https://api.mapx.at/v1/styles/streets"
  }
]
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## MapLibre / Mapbox style

`GET /v1/styles/{id}`

Style JSON (version 8), generated from the style generator. Old ids (`karte`, `karte-morgen`, `karte-abend`, `karte-nacht`, `karte-verkehr`, `luftbild`, `hybrid`, `gelaende`) are aliases until 2027-05-31 (`Deprecation`, `Sunset`, `Link rel=successor-version`).

<details>
<summary>More</summary>

Fonts and sprites point to this API, vector sources to PMTiles on tiles.mapx.at. `ETag`/`Last-Modified`; `If-None-Match` gives 304.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | Style id, optionally with `.json` |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/styles/streets" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles/streets")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "version": 8,
  "name": "streets",
  "sources": {},
  "layers": []
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Sprite of a style

`GET /v1/styles/{id}/{file}`

`file`: `sprite[@2x].json|png` for standard icons, `sprite-sdf[@2x].json|png` for SDF icons. Styles reference this URL.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | Style id |
| `file` | path | string | yes | – | sprite[-sdf][@2x].json\|png |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/styles/streets/sprite.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets/sprite.json");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles/streets/sprite.json")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**: `application/json` · Sprite index (JSON) or atlas (PNG)

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Static Tiles: rendered raster tiles of a style

`GET /v1/styles/{id}/tiles/{tilesize}/{z}/{x}/{y}`

Styles streets*, satellite, satellite-streets, outdoors (old ids as alias). `tilesize` 256 (standard XYZ, map zoom z-1) or 512 (map zoom z). `y` with optional `@2x` and `.png` (default) or `.webp`. Labels are rendered with a margin and clipped. No data or below minimum zoom: 404.

<details>
<summary>More</summary>

Zone `static`. `ETag`, `Cache-Control`.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | Style id |
| `tilesize` | path | string | yes | – | 256 or 512 |
| `z` | path | integer | yes | – | Zoom |
| `x` | path | integer | yes | – | Column |
| `y` | path | string | yes | – | Row[@2x][.png\|.webp] |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420");
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420")
open("result.png", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Example response**: `image/png`, `image/webp` · Image

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `304` | Not modified (with `If-None-Match`) |
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## List fonts

`GET /v1/fonts`

Available font names.

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/fonts" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/fonts");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/fonts")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
[
  {
    "name": "Noto Sans Regular"
  }
]
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Glyph range

`GET /v1/fonts/{fontstack}/{range}.pbf`

`fontstack`: one or more fonts, comma-separated (e.g. `Noto Sans Regular,Open Sans Regular`); the first font containing a character wins. `range`: `{start}-{start+255}`, start divisible by 256 (`0-255`, `256-511`, ...). Response `application/x-protobuf`; `ETag`, `Cache-Control`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `fontstack` | path | string | yes | – | Font name(s), comma-separated |
| `range` | path | string | yes | – | e.g. 0-255 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf" -o result.bin
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf");
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf")
open("result.bin", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Example response**: `application/x-protobuf` · Glyph PBF

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## List sprites

`GET /v1/sprites`

`sprite` (standard icons) and `sprite-sdf` (SDF icons, recolorable).

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/sprites" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/sprites");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/sprites")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**: `application/json` · Sprite list

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Sprite file without style

`GET /v1/sprites/{file}`



**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `file` | path | string | yes | – | sprite[-sdf][@2x].json\|png |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/sprites/sprite.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/sprites/sprite.json");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/sprites/sprite.json")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**: `application/json` · Sprite index (JSON) or atlas (PNG)

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

<Playground group="styles" lang="en" />
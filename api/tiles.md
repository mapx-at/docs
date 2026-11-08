# Tiles API

Tileset list, TileJSON and XYZ tiles (vector, raster, elevation).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Lists the tilesets, describes each as TileJSON and delivers tiles as `{z}/{x}/{y}`. The whole product is also available as one [PMTiles](https://docs.protomaps.com/pmtiles/) file; see [Tilesets](/tilesets/).

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s), `tiles` (100/s) · Guide: [Maps](/guides/maps/)

## List tilesets

`GET /v1/tiles`

Name, aliases, type (vector, raster, dem), format, zoom range, bounds, TileJSON URL, attribution and data date (`stand`) per tileset. Old German names (`basis`, `kataster`, `hoehenlinien`, `relief`, `dop`, `dop_flugblock`) are listed under `aliases` and remain valid until 2027-05-31 (responses carry `Deprecation`, `Sunset` and `Link` headers).

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/tiles" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tiles");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tiles")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "tilesets": [
    {
      "name": "streets",
      "aliases": [
        "basis"
      ],
      "type": "vector",
      "format": "pbf",
      "tilejson": "https://api.mapx.at/v1/tiles/streets.json",
      "minzoom": 0,
      "maxzoom": 14,
      "bounds": [
        9.5,
        46.3,
        17.2,
        49.1
      ],
      "attribution": "...",
      "stand": "2026-10-01"
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## TileJSON 3.0.0

`GET /v1/tiles/{tileset}.json`

TileJSON of a tileset (names from `/v1/tiles`, plus `dtm` and `dsm`). `ETag`/`Last-Modified`, `If-None-Match` gives 304.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `tileset` | path | string | yes | – | Tileset name |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/tiles/streets.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tiles/streets.json");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tiles/streets.json")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "tilejson": "3.0.0",
  "name": "streets",
  "tiles": [
    "https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf"
  ],
  "minzoom": 0,
  "maxzoom": 14,
  "bounds": [
    9.5,
    46.3,
    17.2,
    49.1
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## XYZ tile

`GET /v1/tiles/{tileset}/{z}/{x}/{y}.{ext}`

`ext` must match the tile type: `pbf` (alias `mvt`) for vector, `webp`, `png` or `jpg` for raster. Missing tile or wrong extension: 404 problem+json. Vector tiles are gzip-compressed (`Content-Encoding: gzip`, decompressed if the client does not accept gzip). `ETag` from the file hash, `If-None-Match` gives 304, `Cache-Control: public`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `tileset` | path | string | yes | – | Tileset name, see `/v1/tiles` |
| `z` | path | integer | yes | – | Zoom level |
| `x` | path | integer | yes | – | Tile column |
| `y` | path | integer | yes | – | Tile row |
| `ext` | path | string | yes | – | pbf, mvt, webp, png or jpg |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf");
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf")
open("result.png", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Example response**: `application/vnd.mapbox-vector-tile`, `image/webp`, `image/png`, `image/jpeg` · Tile

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `304` | Not modified (with `If-None-Match`) |
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

<Playground group="tiles" lang="en" />
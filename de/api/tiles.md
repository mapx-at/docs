# Tiles API

Tileset-Liste, TileJSON und XYZ-Kacheln (Vektor, Raster, Höhe).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Listet die Tilesets, beschreibt jedes als TileJSON und liefert Kacheln als `{z}/{x}/{y}`. Jedes Produkt gibt es zusätzlich als [PMTiles](https://docs.protomaps.com/pmtiles/)-Datei, siehe [Tilesets](/de/tilesets/).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `tiles` (100/s) · Guide: [Karten](/de/guides/maps/)

## List tilesets

`GET /v1/tiles`

Name, aliases, type (vector, raster, dem), format, zoom range, bounds, TileJSON URL, attribution and data date (`stand`) per tileset. Old German names (`basis`, `kataster`, `hoehenlinien`, `relief`, `dop`, `dop_flugblock`) are listed under `aliases` and remain valid until 2027-05-31 (responses carry `Deprecation`, `Sunset` and `Link` headers).

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tiles" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tiles", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tiles", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## TileJSON 3.0.0

`GET /v1/tiles/{tileset}.json`

TileJSON of a tileset (names from `/v1/tiles`, plus `dtm` and `dsm`). `ETag`/`Last-Modified`, `If-None-Match` gives 304.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tileset` | path | string | ja | – | Tileset name |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tiles/streets.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tiles/streets.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tiles/streets.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## XYZ tile

`GET /v1/tiles/{tileset}/{z}/{x}/{y}.{ext}`

`ext` must match the tile type: `pbf` (alias `mvt`) for vector, `webp`, `png` or `jpg` for raster. Missing tile or wrong extension: 404 problem+json. Vector tiles are gzip-compressed (`Content-Encoding: gzip`, decompressed if the client does not accept gzip). `ETag` from the file hash, `If-None-Match` gives 304, `Cache-Control: public`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tileset` | path | string | ja | – | Tileset name, see `/v1/tiles` |
| `z` | path | integer | ja | – | Zoom level |
| `x` | path | integer | ja | – | Tile column |
| `y` | path | integer | ja | – | Tile row |
| `ext` | path | string | ja | – | pbf, mvt, webp, png or jpg |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.png", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Beispielantwort**: `application/vnd.mapbox-vector-tile`, `image/webp`, `image/png`, `image/jpeg` · Tile

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `304` | Nicht geändert (mit `If-None-Match`) |
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

<Playground group="tiles" lang="de" />
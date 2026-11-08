# Styles API

Fertige MapLibre-Styles, Schriften, Sprites und gerenderte Raster-Kacheln.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Fertige Styles für MapLibre GL JS und kompatible Clients, dazu Schriften und Sprites. Static Tiles rendern einen Style als Rasterkacheln für Clients ohne Vektorunterstützung.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `tiles` (100/s), `static` (5/s) · Guide: [Karten](/de/guides/maps/)

## List styles

`GET /v1/styles`

Available style ids: streets, streets-dawn, streets-dusk, streets-night, traffic-day, satellite, satellite-streets, outdoors.

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

```json
[
  {
    "id": "streets",
    "url": "https://api.mapx.at/v1/styles/streets"
  }
]
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## MapLibre / Mapbox style

`GET /v1/styles/{id}`

Style JSON (version 8), generated from the style generator. Old ids (`karte`, `karte-morgen`, `karte-abend`, `karte-nacht`, `karte-verkehr`, `luftbild`, `hybrid`, `gelaende`) are aliases until 2027-05-31 (`Deprecation`, `Sunset`, `Link rel=successor-version`).

<details>
<summary>Mehr</summary>

Fonts and sprites point to this API, vector sources to PMTiles on tiles.mapx.at. `ETag`/`Last-Modified`; `If-None-Match` gives 304. With `sources=tilejson` the PMTiles vector/raster sources are replaced by https TileJSON URLs (`https://tiles.mapx.at/v1/tiles/<tileset>.json`), for clients without PMTiles support such as the MapLibre Native offline manager.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Style id, optionally with `.json` |
| `sources` | query | `tilejson` |  | – | `tilejson`: replace pmtiles:// sources by https TileJSON URLs |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles/streets" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles/streets", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

```json
{
  "version": 8,
  "name": "streets",
  "sources": {},
  "layers": []
}
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Sprite of a style

`GET /v1/styles/{id}/{file}`

`file`: `sprite[@2x].json|png` for standard icons, `sprite-sdf[@2x].json|png` for SDF icons. Styles reference this URL.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Style id |
| `file` | path | string | ja | – | sprite[-sdf][@2x].json\|png |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles/streets/sprite.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets/sprite.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles/streets/sprite.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**: `application/json` · Sprite index (JSON) or atlas (PNG)

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Static Tiles: rendered raster tiles of a style

`GET /v1/styles/{id}/tiles/{tilesize}/{z}/{x}/{y}`

Styles streets*, satellite, satellite-streets, outdoors (old ids as alias). `tilesize` 256 (standard XYZ, map zoom z-1) or 512 (map zoom z). `y` with optional `@2x` and `.png` (default) or `.webp`. Labels are rendered with a margin and clipped. No data or below minimum zoom: 404.

<details>
<summary>Mehr</summary>

Zone `static`. `ETag`, `Cache-Control`.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Style id |
| `tilesize` | path | string | ja | – | 256 or 512 |
| `z` | path | integer | ja | – | Zoom |
| `x` | path | integer | ja | – | Column |
| `y` | path | string | ja | – | Row[@2x][.png\|.webp] |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.png", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Beispielantwort**: `image/png`, `image/webp` · Image

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `304` | Nicht geändert (mit `If-None-Match`) |
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## List fonts

`GET /v1/fonts`

Available font names.

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/fonts" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/fonts", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/fonts", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

```json
[
  {
    "name": "Noto Sans Regular"
  }
]
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Glyph range

`GET /v1/fonts/{fontstack}/{range}.pbf`

`fontstack`: one or more fonts, comma-separated (e.g. `Noto Sans Regular,Open Sans Regular`); the first font containing a character wins. `range`: `{start}-{start+255}`, start divisible by 256 (`0-255`, `256-511`, ...). Response `application/x-protobuf`; `ETag`, `Cache-Control`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `fontstack` | path | string | ja | – | Font name(s), comma-separated |
| `range` | path | string | ja | – | e.g. 0-255 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf" -o result.bin
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.bin", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Beispielantwort**: `application/x-protobuf` · Glyph PBF

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## List sprites

`GET /v1/sprites`

`sprite` (standard icons) and `sprite-sdf` (SDF icons, recolorable).

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/sprites" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/sprites", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/sprites", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**: `application/json` · Sprite list

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Sprite file without style

`GET /v1/sprites/{file}`



**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `file` | path | string | ja | – | sprite[-sdf][@2x].json\|png |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/sprites/sprite.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/sprites/sprite.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/sprites/sprite.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**: `application/json` · Sprite index (JSON) or atlas (PNG)

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

<Playground group="styles" lang="de" />
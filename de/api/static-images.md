# Static Images API

Kartenbilder als PNG oder WebP, mit Pins, Pfaden und GeoJSON-Overlays.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Rendert ein Kartenbild für Ausschnitt und Größe, optional mit Overlays. Das Ergebnis ist cachebar (`ETag`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Statische Bilder](/de/guides/static-images/)

## Map image (Static Images, Mapbox form)

`GET /v1/static/{style}/{view}/{size}`

`satellite` (BEV orthophoto, z5 to z18) or `outdoors` (hypsometric tints with hillshade from the DTM, z5 to z16). The `streets*` styles are rendered server-side from the style and the vector tiles (fills, lines, relief shading, place, road and water names; icons from the sprite, house numbers from z16; z0 to z22), usually under 0.2 s per 600x400.

<details>
<summary>Mehr</summary>

`satellite-streets` draws the orthophoto (z5 to z18) with roads, boundaries, contours, names and icons of the vector map on top. Output PNG (default) or WebP, at most 1280x1280 (2560x2560 pixels with @2x). `Cache-Control` and `ETag` (If-None-Match gives 304).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `style` | path | string | ja | – | streets, streets-dawn, streets-dusk, streets-night, satellite, satellite-streets or outdoors (old ids `karte`, `luftbild`, ... as deprecated alias) |
| `view` | path | string | ja | – | lon,lat,zoom[,bearing] (rotation in degrees, pitch only 0) or `auto` (only with overlays, fits the extent) |
| `size` | path | string | ja | – | \{width\}x\{height\}[@2x][.png\|.webp], each 1 to 1280 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/static/satellite/16.3725,48.2085,15/600x400" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/static/satellite/16.3725,48.2085,15/600x400", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/static/satellite/16.3725,48.2085,15/600x400", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Map image with overlays

`GET /v1/static/{style}/{overlay}/{view}/{size}`

`overlay`: comma-separated list (at most 50, 20000 vertices) of `pin-{s|m|l}[-label]+{hex}(lon,lat)` (label not drawn), `path-{width}[+{stroke}[-{opacity}][+{fill}[-{opacity}]]]({polyline, precision 5, URL-encoded})` and `geojson({URL-encoded GeoJSON})` with simplestyle properties (stroke, stroke-width, stroke-opacity, fill, fill-opacity, marker-color, marker-size).

<details>
<summary>Mehr</summary>

Example: `/v1/static/satellite/pin-s+ff0000(15.33,48.22)/15.33,48.22,15/600x400.png`

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `style` | path | string | ja | – | streets, streets-dawn, streets-dusk, streets-night, satellite, satellite-streets or outdoors (old ids `karte`, `luftbild`, ... as deprecated alias) |
| `overlay` | path | string | ja | – | pin-..., path-..., geojson(...), comma-separated |
| `view` | path | string | ja | – | lon,lat,zoom[,bearing] (rotation in degrees, pitch only 0) or `auto` (only with overlays, fits the extent) |
| `size` | path | string | ja | – | \{width\}x\{height\}[@2x][.png\|.webp], each 1 to 1280 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/static/streets/pin-s%2Bff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/static/streets/pin-s%2Bff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/static/streets/pin-s%2Bff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

<Playground group="static-images" lang="de" />
# Kartenstile API: MapLibre-Styles

Source: https://docs.mapx.at/de/api/styles/

> Styles-API: fertige MapLibre-Styles, Schriften (Glyphs), Sprites und gerenderte Rasterkacheln (Static Tiles) für Österreich, mit Parametern und Beispielen.

Fertige Styles für MapLibre GL JS und kompatible Clients, dazu Schriften und Sprites. Static Tiles rendern einen Style als Rasterkacheln für Clients ohne Vektorunterstützung.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `tiles` (100/s), `static` (5/s) · Guide: [Karten](https://docs.mapx.at/de/guides/maps.md)

## Styles auflisten {#list-styles}

`GET /v1/styles`

Verfügbare Style-Ids: streets, streets-dawn, streets-dusk, streets-night, traffic-day, traffic-night, light, dark, satellite, satellite-streets, satellite-current (Sentinel-2 mit Straßen und Namen), outdoors, outdoors-winter, relief, relief-contours, relief-light, relief-dark (nur Gelände: Höhenstufen und Schummerung, gerendert im Browser mit MapLibre GL JS ab 5.6) sowie die 3D-Fassungen outdoors-3d, outdoors-winter-3d, relief-3d, satellite-3d, satellite-streets-3d, streets-3d, streets-night-3d, light-3d, dark-3d (Gelände, Himmel und geneigte Startansicht, die Stadt-Styles mit Gebäudekörpern statt Gelände; nur im Browser).

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/styles", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/styles", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
[
  {
    "id": "streets",
    "url": "https://api.mapx.at/v1/styles/streets"
  }
]
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## MapLibre-/Mapbox-Style {#maplibre--mapbox-style}

`GET /v1/styles/{id}`

Style-JSON (Version 8), erzeugt vom Style-Generator. Schriften und Sprites verweisen auf diese API, Vektorquellen auf PMTiles auf tiles.mapx.at. `ETag`/`Last-Modified`; `If-None-Match` ergibt 304. Mit `sources=tilejson` werden die PMTiles-Vektor- und -Rasterquellen durch https-TileJSON-URLs ersetzt (`https://tiles.mapx.at/v1/tiles/<tileset>.json`), für Clients ohne PMTiles-Unterstützung wie den Offline-Manager von MapLibre Native.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Style-Id, optional mit `.json` |
| `sources` | query | `tilejson` |  | – | `tilejson`: pmtiles://-Quellen durch https-TileJSON-URLs ersetzen |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles/streets" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/styles/streets", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "version": 8,
  "name": "streets",
  "sources": {},
  "layers": []
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Sprite eines Styles {#sprite-of-a-style}

`GET /v1/styles/{id}/{file}`

`file`: `sprite[@2x].json|png` für Standard-Icons, `sprite-sdf[@2x].json|png` für SDF-Icons. Styles verweisen auf diese URL.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Style-Id |
| `file` | path | string | ja | – | sprite[-sdf][@2x].json\|png |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles/streets/sprite.json" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets/sprite.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/styles/streets/sprite.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "airfield": {
    "height": 17,
    "pixelRatio": 1,
    "width": 17,
    "x": 0,
    "y": 0
  },
  "airport": {
    "height": 17,
    "pixelRatio": 1,
    "width": 17,
    "x": 0,
    "y": 17
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Static Tiles: gerenderte Rasterkacheln eines Styles {#static-tiles-rendered-raster-tiles-of-a-style}

`GET /v1/styles/{id}/tiles/{tilesize}/{z}/{x}/{y}`

Styles streets*, light, dark, satellite, satellite-streets, outdoors, outdoors-winter (nicht die Relief-, Verkehrs- und 3D-Styles, sie werden nur im Browser gerendert). `tilesize` 256 (Standard-XYZ, Kartenzoom z-1) oder 512 (Kartenzoom z). `y` mit optionalem `@2x` und `.png` (Standard) oder `.webp`.

Beschriftungen werden mit Rand gerendert und abgeschnitten. Keine Daten oder unter dem Mindestzoom: 404. Zone `static`. `ETag`, `Cache-Control`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Style-Id |
| `tilesize` | path | string | ja | – | 256 oder 512 |
| `z` | path | integer | ja | – | Zoom |
| `x` | path | integer | ja | – | Spalte |
| `y` | path | string | ja | – | Zeile[@2x][.png\|.webp] |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420" -o result.png
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/styles/streets/tiles/512/12/2234/1420", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.png", "wb").write(res.content)
```

**Beispielantwort**: `image/png`, `image/webp` · Bild

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `304` | Nicht geändert (mit `If-None-Match`) |
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Schriften auflisten {#list-fonts}

`GET /v1/fonts`

Verfügbare Schriftnamen.

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/fonts" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/fonts", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/fonts", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
[
  {
    "name": "Noto Sans Regular"
  }
]
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Glyph-Bereich {#glyph-range}

`GET /v1/fonts/{fontstack}/{range}.pbf`

`fontstack`: eine oder mehrere Schriften, durch Komma getrennt (z. B. `Noto Sans Regular,Open Sans Regular`); die erste Schrift, die ein Zeichen enthält, gewinnt. `range`: `{start}-{start+255}`, start durch 256 teilbar (`0-255`, `256-511`, ...). Antwort `application/x-protobuf`; `ETag`, `Cache-Control`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `fontstack` | path | string | ja | – | Schriftname(n), durch Komma getrennt |
| `range` | path | string | ja | – | z. B. 0-255 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf" -o result.pbf
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/fonts/Noto%20Sans%20Regular/0-255.pbf", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.pbf", "wb").write(res.content)
```

**Beispielantwort**: `application/x-protobuf` · Glyph-PBF

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Sprites auflisten {#list-sprites}

`GET /v1/sprites`

`sprite` (Standard-Icons) und `sprite-sdf` (SDF-Icons, umfärbbar).

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/sprites" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/sprites", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/sprites", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
[
  {
    "id": "sprite",
    "url": "https://api.mapx.at/v1/sprites/sprite",
    "files": [
      "https://api.mapx.at/v1/sprites/sprite.json",
      "https://api.mapx.at/v1/sprites/sprite.png",
      "https://api.mapx.at/v1/sprites/sprite@2x.json",
      "https://api.mapx.at/v1/sprites/sprite@2x.png"
    ]
  }
]
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Sprite-Datei ohne Style {#sprite-file-without-style}

`GET /v1/sprites/{file}`

Die gemeinsamen Sprites aller Stile direkt über den Dateinamen (dieselben Dateien wie `/v1/styles/{id}/{file}`): `sprite[@2x].json` ist der Index (Name -&gt; Lage und Größe im Atlas, `pixelRatio`), `sprite[@2x].png` der Atlas; `sprite-sdf` enthält die umfärbbaren SDF-Symbole.

`@2x` für Bildschirme mit hoher Pixeldichte.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `file` | path | string | ja | – | sprite[-sdf][@2x].json\|png |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/sprites/sprite.json" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/sprites/sprite.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/sprites/sprite.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "airfield": {
    "height": 17,
    "pixelRatio": 1,
    "width": 17,
    "x": 0,
    "y": 0
  },
  "airport": {
    "height": 17,
    "pixelRatio": 1,
    "width": 17,
    "x": 0,
    "y": 17
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

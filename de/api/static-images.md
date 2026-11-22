# Static-Images-API: Karte als Bild

Source: https://docs.mapx.at/de/api/static-images/

> Static-Images-API: Kartenbilder als PNG oder WebP mit Pins, Pfaden und GeoJSON-Overlays für Berichte, E-Mails und Exposés, mit Beispielen.

Rendert ein Kartenbild für Ausschnitt und Größe, optional mit Overlays. Das Ergebnis ist cachebar (`ETag`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Statische Bilder](https://docs.mapx.at/de/guides/static-images.md)

## Kartenbild (Static Images, Mapbox-Format) {#map-image-static-images-mapbox-form}

`GET /v1/static/{style}/{view}/{size}`

`satellite` (BEV-Orthofoto, z5 bis z18). Die `streets*`-Styles, `light`, `dark`, `outdoors` und `outdoors-winter` werden am Server aus Style und Vektorkacheln gerendert (Flächen, Linien, Reliefschattierung, Orts-, Straßen- und Gewässernamen; Icons aus dem Sprite, Hausnummern ab z16; z0 bis z22), meist unter 0,2 s pro 600x400.

`satellite-streets` zeichnet das Orthofoto (z5 bis z18) mit Straßen, Grenzen, Höhenlinien, Namen und Icons der Vektorkarte darüber. Ausgabe PNG (Standard) oder WebP, höchstens 1280x1280 (2560x2560 Pixel mit @2x). `Cache-Control` und `ETag` (If-None-Match ergibt 304).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `style` | path | string | ja | – | streets, streets-dawn, streets-dusk, streets-night, light, dark, satellite, satellite-streets, outdoors oder outdoors-winter |
| `view` | path | string | ja | – | lng,lat,zoom[,bearing] (Drehung in Grad, Neigung nur 0) oder `auto` (nur mit Overlays, passt den Ausschnitt an) |
| `size` | path | string | ja | – | \{width\}x\{height\}[@2x][.png\|.webp], je 1 bis 1280 |

**Beispiel**

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

## Kartenbild mit Overlays {#map-image-with-overlays}

`GET /v1/static/{style}/{overlay}/{view}/{size}`

`overlay`: Liste, durch Komma getrennt (höchstens 50, 20000 Stützpunkte), aus `pin-{s|m|l}[-label]+{hex}(lng,lat)` (Beschriftung wird nicht gezeichnet), `path-{width}[+{stroke}[-{opacity}][+{fill}[-{opacity}]]]({polyline, precision 5, URL-encoded})` und `geojson({URL-encoded GeoJSON})` mit simplestyle-Eigenschaften (stroke, stroke-width, stroke-opacity, fill, fill-opacity, marker-color, marker-size).

Beispiel: `/v1/static/satellite/pin-s+ff0000(15.33,48.22)/15.33,48.22,15/600x400.png`

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `style` | path | string | ja | – | streets, streets-dawn, streets-dusk, streets-night, light, dark, satellite, satellite-streets, outdoors oder outdoors-winter |
| `overlay` | path | string | ja | – | pin-..., path-..., geojson(...), durch Komma getrennt |
| `view` | path | string | ja | – | lng,lat,zoom[,bearing] (Drehung in Grad, Neigung nur 0) oder `auto` (nur mit Overlays, passt den Ausschnitt an) |
| `size` | path | string | ja | – | \{width\}x\{height\}[@2x][.png\|.webp], je 1 bis 1280 |

**Beispiel**

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

## Playground

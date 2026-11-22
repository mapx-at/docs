# Tiles-API: TileJSON und XYZ-Kacheln

Source: https://docs.mapx.at/de/api/tiles/

> Tiles-API für Österreich: Tileset-Liste, TileJSON und XYZ-Kacheln als Vektor, Raster und Höhe (Terrain-RGB), mit Parametern und Beispielen.

Listet die Tilesets, beschreibt jedes als TileJSON und liefert Kacheln als `{z}/{x}/{y}`. Alle Produkte: [Tilesets](https://docs.mapx.at/de/tilesets/index.md).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `tiles` (100/s) · Guide: [Karten](https://docs.mapx.at/de/guides/maps.md)

## Tilesets auflisten {#list-tilesets}

`GET /v1/tiles`

Je Tileset Name, Typ (vector, raster, dem), Format, Zoombereich, Grenzen, TileJSON-URL, Attribution, Datenstand (`reference_date`) und Erstellungszeit (`built`). `dtm` wird als Kurzname von `terrain-rgb` angenommen (nicht eigens gelistet).

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tiles" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/tiles", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/tiles", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "tilesets": [
    {
      "name": "streets",
      "type": "vector",
      "format": "pbf",
      "tilejson": "https://tiles.mapx.at/v1/tiles/streets.json",
      "minzoom": 0,
      "maxzoom": 16,
      "bounds": [
        9.5,
        46.37,
        17.17,
        49.02
      ],
      "attribution": "mapx.at | BEV (DKM, Administrative Boundaries, Address Register, Geographical Names), GIP.at, BMLUK (GGN), Umweltbundesamt (CC BY 4.0)",
      "reference_date": "2026-04-01",
      "built": "2026-11-21 11:22:18"
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## TileJSON 3.0.0 {#tilejson-300}

`GET /v1/tiles/{tileset}.json`

TileJSON eines Tilesets (Namen aus `/v1/tiles`, dazu `dtm` und `dsm`). `ETag`/`Last-Modified`, `If-None-Match` ergibt 304.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tileset` | path | string | ja | – | Tileset-Name |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tiles/streets.json" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/tiles/streets.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/tiles/streets.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## XYZ-Kachel {#xyz-tile}

`GET /v1/tiles/{tileset}/{z}/{x}/{y}.{ext}`

`ext` muss zum Kacheltyp passen: `pbf` (Alias `mvt`) für Vektor, `webp` für Raster und Terrain-RGB. Kachel im Zoombereich, aber ohne Daten (z. B. Rand der Abdeckung innerhalb von `bounds`): 204 ohne Body. Falsche Endung oder Zoom außerhalb von `minzoom`/`maxzoom`: 404 problem+json.

Vektorkacheln sind gzip-komprimiert (`Content-Encoding: gzip`, entpackt, wenn der Client kein gzip akzeptiert). `ETag` aus dem Datei-Hash, `If-None-Match` ergibt 304, `Cache-Control: public`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tileset` | path | string | ja | – | Tileset-Name, siehe `/v1/tiles` |
| `z` | path | integer | ja | – | Zoomstufe |
| `x` | path | integer | ja | – | Kachelspalte |
| `y` | path | integer | ja | – | Kachelzeile |
| `ext` | path | string | ja | – | pbf oder mvt (Vektor), webp (Raster) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf" -o result.pbf
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/tiles/streets/14/8937/5681.pbf", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.pbf", "wb").write(res.content)
```

**Beispielantwort**: `application/vnd.mapbox-vector-tile`, `image/webp` · Kachel

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `204` | Keine Daten für diese Kachel (leer) |
| `304` | Nicht geändert (mit `If-None-Match`) |
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

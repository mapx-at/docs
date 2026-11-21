# Höhen-API: Höhe und Höhenprofil

Source: https://docs.mapx.at/de/api/elevation/

> Höhen-API für Österreich: Geländehöhe und Oberflächenhöhe für Punkte sowie Höhenprofile entlang einer Linie, mit Parametern und Beispielanfragen.

Höhe über Meer (m) für Punkte oder entlang einer Linie, aus dem Geländemodell (`dtm`) oder dem Oberflächenmodell (`dsm`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `elevation` (10/s), `search` (10/s) · Guide: [Höhe und Gelände](https://docs.mapx.at/de/guides/elevation.md)

## Höhen (bilinear aus Terrain-RGB z16, 0,1 m) {#elevations-bilinear-from-z16-terrain-rgb-01-m}

`GET /v1/elevation`

Entweder `points` (bis zu 512 Punkte) oder `path` (Linie auf der Kugel, nach Länge gleichmäßig in `samples` Punkte geteilt). Außerhalb des Gebiets ist `elevation` null. Eine POST-Form gibt es nicht, und das Gateway begrenzt die Anfrage-URL auf etwa 8 KB (längere URLs bekommen 414), ein GET trägt daher praktisch etwa 400 Punkte mit 6 Nachkommastellen, mit mehr Stellen weniger.

Größere Punktmengen auf mehrere Anfragen aufteilen; für Punkte entlang einer Linie nur die Stützpunkte als `path` senden und `samples` (bis 1024) die Abtastung überlassen, oder `/v1/elevation/profile` verwenden.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `points` | query | string |  | – | lon,lat;lon,lat;... (höchstens 512; die URL-Grenze von etwa 8 KB erlaubt meist weniger) |
| `path` | query | string |  | – | lon,lat;lon,lat;... (2 bis 512 Stützpunkte) |
| `samples` | query | integer (2–1024) |  | `100` | Nur mit path |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (Gelände) oder dsm (Oberfläche) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3725,
          48.2083
        ]
      },
      "properties": {
        "elevation": 171.3,
        "resolution_m": 1,
        "model": "dtm",
        "reference_date": "2025-09-15"
      }
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Höhe an einem Punkt (DTM und DSM) {#height-at-a-point-dtm-and-dsm}

`GET /v1/elevation/{coordinate}`

Ein Punktobjekt; properties: `dtm` (Gelände), `dsm` (Oberfläche, null, wenn nicht verfügbar), `resolution_m`, `reference_date`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat oder lon,lat.json |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation/12.694,47.074" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/elevation/12.694,47.074", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/elevation/12.694,47.074", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "Feature",
  "geometry": {
    "type": "Point",
    "coordinates": [
      16.3725,
      48.2083
    ]
  },
  "properties": {
    "dtm": 171.3,
    "dsm": 171.9,
    "resolution_m": 1,
    "reference_date": "2025-09-15"
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Höhenprofil entlang einer Linie {#elevation-profile-along-a-line}

`GET /v1/elevation/profile`

Eine GeoJSON-Linie mit z-Koordinate; properties: `distances`, `elevations`, `ascent_m`, `descent_m`, `min_m`, `max_m`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | query | string | ja | – | lon,lat;lon,lat;... (2 bis 512 Punkte) |
| `samples` | query | string |  | `100` | 2 bis 1024, Standard 100 |
| `model` | query | string |  | – | dtm\|dsm |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "Feature",
  "geometry": {
    "type": "LineString",
    "coordinates": [
      [
        16.37,
        48.21,
        171
      ],
      [
        16.38,
        48.22,
        180.5
      ]
    ]
  },
  "properties": {
    "distances": [
      0,
      1400
    ],
    "elevations": [
      171,
      180.5
    ],
    "ascent_m": 9.5,
    "descent_m": 0,
    "min_m": 171,
    "max_m": 180.5
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

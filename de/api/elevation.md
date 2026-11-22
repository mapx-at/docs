# Höhen-API: Höhe und Höhenprofil

Source: https://docs.mapx.at/de/api/elevation/

> Höhen-API für Österreich: Geländehöhe und Oberflächenhöhe für Punkte sowie Höhenprofile entlang einer Linie, mit Parametern und Beispielanfragen.

Höhe über Meer (m) für Punkte oder entlang einer Linie, aus dem Geländemodell (`dtm`) oder dem Oberflächenmodell (`dsm`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `elevation` (10/s), `search` (10/s) · Guide: [Höhe und Gelände](https://docs.mapx.at/de/guides/elevation.md)

## Höhen (bilinear aus Terrain-RGB z16, 0,1 m) {#elevations-bilinear-from-z16-terrain-rgb-01-m}

`GET /v1/elevation`

Entweder `points` (bis zu 512 Punkte) oder `path` (Linie auf der Kugel, nach Länge gleichmäßig in `samples` Punkte geteilt). Außerhalb des Gebiets ist `elevation` null. Das Gateway begrenzt die Anfrage-URL auf etwa 8 KB (längere URLs bekommen 414 als problem+json), ein GET trägt daher praktisch etwa 400 Punkte mit 6 Nachkommastellen, mit mehr Stellen weniger.

Größere Punktmengen auf mehrere Anfragen aufteilen; für Punkte entlang einer Linie nur die Stützpunkte als `path` senden und `samples` (bis 1024) die Abtastung überlassen, oder die Linie an `POST /v1/elevation/profile` schicken (bis 20000 Stützpunkte).

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
        "resolution": 1,
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

Ein Punktobjekt; properties: `dtm` (Gelände), `dsm` (Oberfläche, null, wenn nicht verfügbar), `resolution` (Meter), `reference_date`.

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
    "resolution": 1,
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

Eine GeoJSON-Linie; properties: `distance`, `distances`, `elevations`, `ascent`, `descent`, `ascent_threshold`, `min_elevation`, `max_elevation` (alle in Metern), `resolution`, `model`, `reference_date`. Anstieg und Abstieg werden mit Hysterese gezählt: eine Steigung oder ein Gefälle zählt erst, wenn sich die Höhe um mindestens `ascent_threshold` (3 m) vom letzten Wendepunkt entfernt hat.

Die Summen hängen dadurch kaum von `samples` ab, und Geländerauschen wird nicht aufsummiert (flaches Marchfeld, 9,6 km: 0 m statt 46 m ungefiltert; Großglockner-Hochalpenstraße ab Heiligenblut: rund 1150 m). Mit `model=dsm` zählen Gebäude und Bewuchs als Gelände, der Anstieg fällt höher aus. Die Linie hat z-Koordinaten, wenn jeder Abtastpunkt eine Höhe hat; sonst sind alle Positionen 2D und die Lücken stehen als `null` in `elevations`. Für eine Routengeometrie die POST-Form verwenden.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | query | string | ja | – | lon,lat;lon,lat;... (2 bis 512 Punkte) |
| `samples` | query | integer (2–1024) |  | `100` | 2 bis 1024, Standard 100 |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (Gelände, Standard) oder dsm (Oberfläche) |

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
    "ascent": 9.5,
    "descent": 0,
    "ascent_threshold": 3,
    "min_elevation": 171,
    "max_elevation": 180.5
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

## Höhenprofil für eine Linie im Körper (z. B. eine Routengeometrie) {#elevation-profile-for-a-line-in-the-body-eg-a-route-geometry}

`POST /v1/elevation/profile`

Antwort wie bei GET. Körper: ein GeoJSON-LineString, ein Feature mit LineString (z. B. `routes[0].geometry` aus `/v1/directions?geometries=geojson&overview=full`) oder `{"coordinates": [[lon, lat], ...]}`, 2 bis 20000 Stützpunkte; `samples` und `model` im Körper oder als Query-Parameter.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `samples` | query | integer (2–1024) |  | `100` | 2 bis 1024, Standard 100 |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (Standard) oder dsm |

**Request-Body (JSON)**

```json
{
  "type": "LineString",
  "coordinates": [
    [
      12.8418,
      47.0378
    ],
    [
      12.7865,
      47.0739
    ],
    [
      12.7303,
      47.0792
    ]
  ]
}
```

**Beispiel**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/elevation/profile" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"LineString","coordinates":[[12.8418,47.0378],[12.7865,47.0739],[12.7303,47.0792]]}' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/elevation/profile", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify({
    "type": "LineString",
    "coordinates": [
      [
        12.8418,
        47.0378
      ],
      [
        12.7865,
        47.0739
      ],
      [
        12.7303,
        47.0792
      ]
    ]
  }),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/elevation/profile", headers={"Authorization": "Bearer YOUR_TOKEN"}, json={
    "type": "LineString",
    "coordinates": [
      [
        12.8418,
        47.0378
      ],
      [
        12.7865,
        47.0739
      ],
      [
        12.7303,
        47.0792
      ]
    ]
  })
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
    "ascent": 9.5,
    "descent": 0,
    "ascent_threshold": 3,
    "min_elevation": 171,
    "max_elevation": 180.5
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

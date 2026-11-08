# Elevation API

Gelände- und Oberflächenhöhe für Punkte und Höhenprofile entlang einer Linie.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Höhe über Meer (m) für Punkte oder entlang einer Linie, aus dem Geländemodell (`dtm`) oder dem Oberflächenmodell (`dsm`).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `elevation` (10/s), `search` (10/s) · Guide: [Höhe und Gelände](/de/guides/elevation/)

## Elevations (bilinear from z16 Terrain-RGB, 0.1 m)

`GET /v1/elevation`

Either `points` (up to 512 points) or `path` (polyline on the sphere, split evenly by length into `samples` points). Outside the area `elevation` is null.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `points` | query | string |  | – | lon,lat;lon,lat;... (at most 512) |
| `path` | query | string |  | – | lon,lat;lon,lat;... (2 to 512 vertices) |
| `samples` | query | integer (2–1024) |  | `100` | With path only |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (terrain) or dsm (surface) |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm")
data = res.json()
```

</TabItem>
</Tabs>

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Height at a point (DTM and DSM)

`GET /v1/elevation/{coordinate}`

One point feature; properties: `dtm` (terrain), `dsm` (surface, null if unavailable), `resolution_m`, `reference_date` (`stichtag` deprecated until 2027-05-31).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat or lon,lat.json |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/elevation/12.694,47.074" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/elevation/12.694,47.074");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/elevation/12.694,47.074")
data = res.json()
```

</TabItem>
</Tabs>

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Elevation profile along a line

`GET /v1/elevation/profile`

One GeoJSON line with z coordinate; properties: `distances`, `elevations`, `ascent_m`, `descent_m`, `min_m`, `max_m`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | query | string | ja | – | lon,lat;lon,lat;... (2 to 512 points) |
| `samples` | query | string |  | `100` | 2 to 1024, default 100 |
| `model` | query | string |  | – | dtm\|dsm |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20")
data = res.json()
```

</TabItem>
</Tabs>

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
  …
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

<Playground group="elevation" lang="de" />
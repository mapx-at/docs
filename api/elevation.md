# Elevation API

Terrain and surface height for points and elevation profiles along a line.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Height above sea level (m) for points or along a line, from the terrain model (`dtm`) or the surface model (`dsm`).

Base URL: `https://api.mapx.at` · Rate-limit zone: `elevation` (10/s), `search` (10/s) · Guide: [Elevation and terrain](/guides/elevation/)

## Elevations (bilinear from z16 Terrain-RGB, 0.1 m)

`GET /v1/elevation`

Either `points` (up to 512 points) or `path` (polyline on the sphere, split evenly by length into `samples` points). Outside the area `elevation` is null.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `points` | query | string |  | – | lon,lat;lon,lat;... (at most 512) |
| `path` | query | string |  | – | lon,lat;lon,lat;... (2 to 512 vertices) |
| `samples` | query | integer (2–1024) |  | `100` | With path only |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (terrain) or dsm (surface) |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Height at a point (DTM and DSM)

`GET /v1/elevation/{coordinate}`

One point feature; properties: `dtm` (terrain), `dsm` (surface, null if unavailable), `resolution_m`, `reference_date` (`stichtag` deprecated until 2027-05-31).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinate` | path | string | yes | – | lon,lat or lon,lat.json |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation/12.694,47.074" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/elevation/12.694,47.074", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/elevation/12.694,47.074", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Elevation profile along a line

`GET /v1/elevation/profile`

One GeoJSON line with z coordinate; properties: `distances`, `elevations`, `ascent_m`, `descent_m`, `min_m`, `max_m`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | query | string | yes | – | lon,lat;lon,lat;... (2 to 512 points) |
| `samples` | query | string |  | `100` | 2 to 1024, default 100 |
| `model` | query | string |  | – | dtm\|dsm |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=20", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

<Playground group="elevation" lang="en" />
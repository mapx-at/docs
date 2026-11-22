# Elevation API: height and profile

Source: https://docs.mapx.at/api/elevation/

> Elevation API for Austria: terrain height and surface height for points and elevation profiles along a line, with parameters and example requests.

Height above sea level (m) for points or along a line, from the terrain model (`dtm`) or the surface model (`dsm`).

Base URL: `https://api.mapx.at` · Rate-limit zone: `elevation` (10/s), `search` (10/s) · Guide: [Elevation and terrain](https://docs.mapx.at/guides/elevation.md)

## Elevations (bilinear from z16 Terrain-RGB, 0.1 m)

`GET /v1/elevation`

Either `points` (up to 512 points) or `path` (polyline on the sphere, split evenly by length into `samples` points). Outside the area `elevation` is null. The gateway limits the request URL to about 8 KB (longer URLs get 414 as problem+json), so in practice a GET carries about 400 points with 6 decimals, fewer with more digits.

Split larger point sets into several requests; for points along a line send only the vertices as `path` and let `samples` (up to 1024) do the sampling, or send the line to `POST /v1/elevation/profile` (up to 20000 vertices).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `points` | query | string |  | – | lon,lat;lon,lat;... (at most 512; the URL limit of about 8 KB usually allows fewer) |
| `path` | query | string |  | – | lon,lat;lon,lat;... (2 to 512 vertices) |
| `samples` | query | integer (2–1024) |  | `100` | With path only |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (terrain) or dsm (surface) |

**Example**

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Height at a point (DTM and DSM)

`GET /v1/elevation/{coordinate}`

One point feature; properties: `dtm` (terrain), `dsm` (surface, null if unavailable), `resolution_m`, `reference_date`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinate` | path | string | yes | – | lon,lat or lon,lat.json |

**Example**

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Elevation profile along a line

`GET /v1/elevation/profile`

One GeoJSON line; properties: `distances`, `elevations`, `ascent_m`, `descent_m`, `ascent_threshold_m`, `min_m`, `max_m`. Ascent and descent are counted with hysteresis: a rise or fall counts only once the elevation has moved at least `ascent_threshold_m` (3 m) from the last turning point, so the totals hardly depend on `samples` and terrain noise is not summed up (flat Marchfeld, 9.6 km: 0 m instead of 46 m unfiltered; Grossglockner High Alpine Road up from Heiligenblut: about 1150 m).

With `model=dsm` buildings and vegetation count as terrain, so ascent is higher. The line has z coordinates when every sample has an elevation; otherwise all positions are 2D and the gaps are `null` in `elevations`. For a route geometry use the POST form.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | query | string | yes | – | lon,lat;lon,lat;... (2 to 512 points) |
| `samples` | query | integer (2–1024) |  | `100` | 2 to 1024, default 100 |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (terrain, default) or dsm (surface) |

**Example**

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
    "ascent_threshold_m": 3,
    "min_m": 171,
    "max_m": 180.5
  }
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Elevation profile for a line in the body (e.g. a route geometry)

`POST /v1/elevation/profile`

Same response as GET. Body: a GeoJSON LineString, a Feature with a LineString (e.g. `routes[0].geometry` from `/v1/directions?geometries=geojson&overview=full`) or `{"coordinates": [[lon, lat], ...]}`, 2 to 20000 vertices; `samples` and `model` in the body or as query parameters.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `samples` | query | integer (2–1024) |  | `100` | 2 to 1024, default 100 |
| `model` | query | `dtm` \| `dsm` |  | `dtm` | dtm (default) or dsm |

**Request body (JSON)**

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

**Example**

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
    "ascent_threshold_m": 3,
    "min_m": 171,
    "max_m": 180.5
  }
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

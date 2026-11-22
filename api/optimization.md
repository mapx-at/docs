# Route optimization API

Source: https://docs.mapx.at/api/optimization/

> Route optimization API: best order for up to 25 stops and tour planning for several vehicles, with parameters and example requests.

Finds the best order of stops (traveling salesman) or plans tours for several vehicles with capacities and time windows.

Base URL: `https://api.mapx.at` · Rate-limit zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/guides/navigation.md)

## Optimized order

`GET /v1/optimized-trips/{profile}/{coordinates}`

2 to 25 coordinates. Exact up to 12 points, above that nearest neighbour with 2-opt and Or-opt over the duration matrix. Response like Directions: `trips[]` with one route, `waypoints[]` in input order with `waypoint_index` (position in the trip) and `trips_index`.

Expensive: zone `heavy`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (avoids active closures from the EVIS/ASFINAG reports and slows motorway sections where variable message signs show a lowered limit; no area-wide live traffic), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking |
| `coordinates` | path | string | yes | – | lon,lat;lon,lat;... |
| `source` | query | string |  | – | any (default) or first (point 0 is the start) |
| `destination` | query | string |  | – | any (default) or last (last point is the end) |
| `roundtrip` | query | string |  | – | true (default) returns to the start, or false; true with destination=last needs source=first |
| `steps` | query | string |  | – | true for turn-by-turn steps |
| `geometries` | query | string |  | – | geojson\|polyline\|polyline6 |
| `overview` | query | string |  | – | simplified (default) \| full \| false |
| `language` | query | string |  | – | de (default) or en; other values 400 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "code": "Ok",
  "trips": [],
  "waypoints": []
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Tour planning (simplified Optimization v2)

`POST /v1/optimization`

Vehicles with start/end/capacity/time window, services (one location) and shipments (pickup before delivery, same vehicle) with duration, time windows and amounts. Heuristic: regret insertion, local search, ruin and recreate until `options.time_limit_ms`. At most 10 vehicles, 50 jobs, 60 distinct locations, 256 KB.

Needs a secret token (`mx_sk_`); public tokens get `403 pk-not-allowed`, so the playground cannot run it.

Times in seconds. Expensive: zone `heavy`.

**Request body (JSON)**

| Name | Type | Description |
|---|---|---|
| `profile` | string | driving (default), cycling, cycling-mtb or walking |

**Example**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/optimization" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"vehicles":[{"id":1,"start":[16.3725,48.2083],"end":[16.3725,48.2083]}],"services":[{"id":1,"location":[15.6256,48.2049],"service":300},{"id":2,"location":[14.2858,48.3069],"service":300}]}' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/optimization", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify({
    "vehicles": [
      {
        "id": 1,
        "start": [
          16.3725,
          48.2083
        ],
        "end": [
          16.3725,
          48.2083
        ]
      }
    ],
    "services": [
      {
        "id": 1,
        "location": [
          15.6256,
          48.2049
        ],
        "service": 300
      },
      {
        "id": 2,
        "location": [
          14.2858,
          48.3069
        ],
        "service": 300
      }
    ]
  }),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/optimization", headers={"Authorization": "Bearer YOUR_TOKEN"}, json={
    "vehicles": [
      {
        "id": 1,
        "start": [
          16.3725,
          48.2083
        ],
        "end": [
          16.3725,
          48.2083
        ]
      }
    ],
    "services": [
      {
        "id": 1,
        "location": [
          15.6256,
          48.2049
        ],
        "service": 300
      },
      {
        "id": 2,
        "location": [
          14.2858,
          48.3069
        ],
        "service": 300
      }
    ]
  })
data = res.json()
```

**Example response**

```json
{
  "summary": {
    "cost": 9000,
    "unassigned": 0
  },
  "routes": [],
  "unassigned": []
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `405` | Method not allowed |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

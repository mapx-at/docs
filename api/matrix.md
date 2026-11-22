# Matrix API: travel times and distances

Source: https://docs.mapx.at/api/matrix/

> Matrix API for Austria: travel times and distances between many origins and destinations in one request, with parameters and examples.

Durations (seconds) and distances (meters) between all pairs of coordinates, or only between `sources` and `destinations`. Profiles: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

Base URL: `https://api.mapx.at` · Rate-limit zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/guides/navigation.md)

## Travel time / distance matrix

`GET /v1/matrix/{profile}/{coordinates}`

2 to 25 coordinates. `sources`/`destinations` as index list (`0;2`) or `all`. Zone `heavy`. Closures: driving, driving-traffic and driving-truck compute with the same closures and weights as directions (binding closures never, other full closures such as winter closures only as a last resort, conditional regulations only with driving-traffic), so each entry matches the duration and distance of a directions request between the same two points.

`closures` reports whether closures were applied (`applied`, `count`, `binding`, `binding_count`, `fetched`); cycling and walking ignore closures and have no `closures`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (avoids active closures from the EVIS/ASFINAG reports and slows motorway sections where variable message signs show a lowered limit; no area-wide live traffic), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking |
| `coordinates` | path | string | yes | – | lng,lat;lng,lat;... |
| `sources` | query | string |  | – | Indices, semicolon-separated |
| `destinations` | query | string |  | – | Indices, semicolon-separated |
| `annotations` | query | string |  | – | duration (default), distance or both |
| `weight` | query | string |  | `40` | driving-truck only: gross weight in t (default 40, 0.5 to 60); avoids weight-class bans and weight limits |
| `height` | query | string |  | `4` | driving-truck only: vehicle height in m (default 4, 1.5 to 5); avoids height limits |
| `width` | query | string |  | `2.55` | driving-truck only: vehicle width in m (default 2.55, 1 to 3.5) |
| `length` | query | string |  | – | driving-truck only: vehicle length in m (2 to 40); length limits are only checked when given |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | driving-truck only: semi (articulated lorry, default), rigid (lorry without trailer) or trailer (lorry with trailer) |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "durations": [
    [
      0,
      2890.4
    ],
    [
      2901.2,
      0
    ]
  ],
  "sources": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ]
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ]
    }
  ],
  "destinations": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ]
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ]
    }
  ],
  "closures": {
    "applied": true,
    "count": 113,
    "binding": true,
    "binding_count": 8,
    "fetched": "2026-11-22T01:39:31Z"
  }
}
```

| Field | Type | Description |
|---|---|---|
| `durations` | array | seconds |
| `distances` | array | meters |
| `closures` | object | Road closures taken into account (driving, driving-traffic, driving-truck only; absent for cycling and walking). |

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

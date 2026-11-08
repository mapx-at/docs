# Optimization API

Beste Reihenfolge für bis zu 25 Stopps und Tourenplanung für mehrere Fahrzeuge.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Findet die beste Reihenfolge von Stopps (Handlungsreisender) oder plant Touren für mehrere Fahrzeuge mit Kapazitäten und Zeitfenstern.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Navigation](/de/guides/navigation/)

## Optimized order

`GET /v1/optimized-trips/{profile}/{coordinates}`

2 to 25 coordinates. Exact up to 12 points, above that nearest neighbour with 2-opt and Or-opt over the duration matrix. Response like Directions: `trips[]` with one route, `waypoints[]` in input order with `waypoint_index` (position in the trip) and `trips_index`.

<details>
<summary>Mehr</summary>

Expensive: zone `heavy`.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | ja | – | lon,lat;lon,lat;... |
| `source` | query | string |  | – | any (default) or first (point 0 is the start) |
| `destination` | query | string |  | – | any (default) or last (last point is the end) |
| `roundtrip` | query | string |  | – | true (default) returns to the start, or false; true with destination=last needs source=first |
| `steps` | query | string |  | – | true for turn-by-turn steps |
| `geometries` | query | string |  | – | geojson\|polyline\|polyline6 |
| `overview` | query | string |  | – | simplified (default) \| full \| false |
| `language` | query | string |  | – | de (default) or en |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

```json
{
  "code": "Ok",
  "trips": [],
  "waypoints": []
}
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Tour planning (simplified Optimization v2)

`POST /v1/optimization`

Vehicles with start/end/capacity/time window, services (one location) and shipments (pickup before delivery, same vehicle) with duration, time windows and amounts. Heuristic: regret insertion, local search, ruin and recreate until `options.time_limit_ms`. At most 10 vehicles, 50 jobs, 60 distinct locations, 256 KB.

<details>
<summary>Mehr</summary>

Times in seconds. Expensive: zone `heavy`.

</details>

**Request-Body (JSON)**

| Name | Typ | Beschreibung |
|---|---|---|
| `profile` | string | driving (default), cycling, cycling-mtb or walking |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -X POST "https://api.mapx.at/v1/optimization" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"vehicles":[{"id":1,"start":[16.3725,48.2083],"end":[16.3725,48.2083]}],"services":[{"id":1,"location":[15.6256,48.2049],"service":300},{"id":2,"location":[14.2858,48.3069],"service":300}]}' | jq
```

</TabItem>
<TabItem label="JavaScript">

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

</TabItem>
<TabItem label="Python">

```python

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

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `405` | Methode nicht erlaubt |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

<Playground group="optimization" lang="de" />
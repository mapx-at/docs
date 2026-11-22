# Isochrone API: reachable areas

Source: https://docs.mapx.at/api/isochrone/

> Isochrone API for Austria: areas reachable within given minutes or meters by car, truck, bike, foot or public transport, with parameters and examples.

Polygons of the area reachable from a point within the given times or distances. Profiles: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`, plus `transit`.

Base URL: `https://api.mapx.at` · Rate-limit zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/guides/navigation.md)

## Isochrones

`GET /v1/isochrone/{profile}/{coordinate}`

Reachable areas, largest first; `properties.contour/color/opacity`. Zone `heavy`. Profile `transit` computes walking plus public transport (GTFS timetable, RAPTOR): walk to stops, ride from `depart_at`, walk on from every stop reached; only `contours_minutes` (default 5,10,15, at most 120 min), the timetable's validity period applies (otherwise 422), `metadata` carries `depart_at` (local time with offset) and `transit_stops`.

`access=foot|bike|car` (default `foot`) chooses how the first stop is reached: `bike` = Bike & Ride (bike to any stop, +2 min for locking), `car` = Park & Ride (drive, at most 25 min, to stops with rail/metro/tram service, +3 min for parking, and to designated GIP P+R sites, +2 min and a walk to the stop); the fastest combination wins, exit is always on foot. `park=official` restricts parking to designated P+R/B+R sites. `metadata` then adds `access`, `park`, `access_stops_improved` and `access_sites` (top `sites` entries: `source` site|stop, `kind`, `name`, `lng`, `lat`, `arrive_min`, `ready_min`, `stops_won`). Bike carriage on trains is not in the timetable data, so there is no `egress=bike`. The other profiles reject `depart_at`, `access` and `park` (400). Closures: driving, driving-traffic and driving-truck do not extend across active full closures from the traffic reports (the same ones directions avoids, see `/v1/traffic/incidents` with `closed: true`), including closures without a binding note such as winter closures of pass roads: an isochrone has no destination behind the closure, so there is no last resort; the road the start point snaps to stays usable. `closures` reports whether closures were applied (`applied`, `count`, `binding`, `binding_count`, `fetched`), `closures_avoided` lists the closures at which the largest contour stops (`id`, `text`, `road`, `category`, `binding`), both at the top level as with directions and matrix.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking or transit (walking plus public transport from `depart_at`) |
| `coordinate` | path | string | yes | – | lng,lat |
| `contours_minutes` | query | string |  | `5,10,15` | Up to 4 values, comma-separated (default 5,10,15) |
| `contours_meters` | query | string |  | – | Distances instead of minutes (not for `transit`) |
| `depart_at` | query | string |  | `now` | Only `transit`: departure, ISO 8601 local time Europe/Vienna without offset (`2026-11-10T08:00`), or with `Z`/offset, or Unix seconds; default now |
| `access` | query | string |  | – | Only `transit`: foot (default) \| bike (Bike & Ride) \| car (Park & Ride) |
| `park` | query | string |  | `bike every stop` | Only `transit` with bike/car: any (default: bike every stop, car rail stops) \| official (only designated P+R/B+R sites) |
| `sites` | query | string |  | `10` | Only `transit` with bike/car: number of access entries in `metadata.access_sites` (default 10, max 50) |
| `weight` | query | string |  | `40` | driving-truck only: gross weight in t (default 40, 0.5 to 60); avoids weight-class bans and weight limits |
| `height` | query | string |  | `4` | driving-truck only: vehicle height in m (default 4, 1.5 to 5); avoids height limits |
| `width` | query | string |  | `2.55` | driving-truck only: vehicle width in m (default 2.55, 1 to 3.5) |
| `length` | query | string |  | – | driving-truck only: vehicle length in m (2 to 40); length limits are only checked when given |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | driving-truck only: semi (articulated lorry, default), rigid (lorry without trailer) or trailer (lorry with trailer) |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response** (shortened: lists show only their first entries)

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "contour": 15,
        "metric": "time",
        "color": "#bf4040",
        "opacity": 0.33
      },
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
          [
            [
              [
                16.3,
                48.2
              ],
              [
                16.4,
                48.2
              ]
            ]
          ]
        ]
      }
    }
  ]
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

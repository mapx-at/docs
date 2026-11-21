# Transit API: public transport Austria

Source: https://docs.mapx.at/api/transit/

> Transit API for public transport in Austria: journeys, stops and departure boards from the timetable data, with parameters and examples.

Journeys with public transport (on foot or by bike to the stop), departure boards and stops, based on the scheduled timetable.

Base URL: `https://api.mapx.at` · Rate-limit zone: `route` (5/s), `search` (10/s) · Guide: [Transit departures](https://docs.mapx.at/guides/transit.md)

## Public transit: walking plus transit (GTFS timetable, RAPTOR)

`GET /v1/directions/transit/{coordinates}`

Exactly origin and destination. Connections with walks on the GIP.at network and rides (line, stops, times). Time via `depart_at` (default now) or `arrive_by`, ISO 8601 in Vienna local time. Timetable data: OeBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitaetsverbuende Oesterreich OG, data.mobilitaetsverbuende.at.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | path | string | yes | – | lon,lat;lon,lat (exactly origin and destination) |
| `depart_at` | query | string |  | – | Earliest departure, e.g. 2026-11-09T08:00 |
| `arrive_by` | query | string |  | – | Latest arrival, instead of depart_at |
| `routes` | query | string |  | `3` | 1 to 5, default 3 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response** (shortened: lists show only their first entries)

```json
{
  "code": "Ok",
  "routes": [
    {
      "arrival": "2026-11-23T08:12:51+01:00",
      "departure": "2026-11-23T08:00:26+01:00",
      "distance": 4801,
      "duration": 745,
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3725,
            48.2085
          ]
        ]
      },
      "legs": [
        {
          "arrival": "2026-11-23T08:02:00+01:00",
          "departure": "2026-11-23T08:00:26+01:00",
          "distance": 117,
          "duration": 94,
          "from": {
            "location": [
              16.3725,
              48.2085
            ],
            "name": null
          },
          "mode": "walking",
          "summary": "walking",
          "to": {
            "location": [
              16.371607,
              48.208086
            ],
            "name": "Wien Stephansplatz"
          }
        }
      ],
      "summary": "U3",
      "transfers": 0,
      "weight": 745,
      "weight_name": "duration"
    }
  ],
  "query": {
    "depart_at": "2026-11-23T08:00:00+01:00",
    "profile": "transit"
  },
  "attribution": "mapx.at; Fahrplandaten: OeBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)"
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Bike to the stop plus transit

`GET /v1/directions/bike_transit/{coordinates}`

Like transit, but the first and last leg by bike (mode `cycling`, up to 15 min each); the bike is not taken on board.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | path | string | yes | – | lon,lat;lon,lat (exactly origin and destination) |
| `depart_at` | query | string |  | – | Earliest departure |
| `arrive_by` | query | string |  | – | Latest arrival |
| `routes` | query | string |  | `3` | 1 to 5, default 3 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/bike_transit/16.3725,48.2085;16.4202,48.1855" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/bike_transit/16.3725,48.2085;16.4202,48.1855", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/bike_transit/16.3725,48.2085;16.4202,48.1855", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response** (shortened: lists show only their first entries)

```json
{
  "code": "Ok",
  "routes": [
    {
      "arrival": "2026-11-23T08:12:51+01:00",
      "departure": "2026-11-23T08:00:26+01:00",
      "distance": 4801,
      "duration": 745,
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3725,
            48.2085
          ]
        ]
      },
      "legs": [
        {
          "arrival": "2026-11-23T08:02:00+01:00",
          "departure": "2026-11-23T08:00:26+01:00",
          "distance": 117,
          "duration": 94,
          "from": {
            "location": [
              16.3725,
              48.2085
            ],
            "name": null
          },
          "mode": "walking",
          "summary": "walking",
          "to": {
            "location": [
              16.371607,
              48.208086
            ],
            "name": "Wien Stephansplatz"
          }
        }
      ],
      "summary": "U3",
      "transfers": 0,
      "weight": 745,
      "weight_name": "duration"
    }
  ],
  "query": {
    "depart_at": "2026-11-23T08:00:00+01:00",
    "profile": "transit"
  },
  "attribution": "mapx.at; Fahrplandaten: OeBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)"
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Departure board (scheduled GTFS timetable)

`GET /v1/departures`

Next departures at a station (id from `/v1/stops`, e.g. `at:43:7226`) or at all stations around a coordinate. Service days follow calendar and feed period, trips past midnight included. Per departure: time, line (name, mode, route_type), headsign, destination, platform (null if unknown), operator, stop.

Times come from the scheduled timetable: `realtime` is always false. Outside the feed period 422, unknown station 404. Timetable data: OeBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitaetsverbuende Oesterreich OG, data.mobilitaetsverbuende.at.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `stop` | query | string | yes | – | Station id (IFOPT `at:<state>:<nr>`, a platform id such as `at:43:7226:0:2` is merged into its station) or lon,lat |
| `time` | query | string |  | `now` | Start time, ISO 8601 (Vienna local) or Unix seconds, default now |
| `limit` | query | string |  | `20` | 1 to 100, default 20 |
| `radius` | query | string |  | `300` | With lon,lat only: meters, 1 to 2000, default 300 |
| `horizon` | query | string |  | `14400` | Window in seconds, 60 to 86400, default 14400 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response** (shortened: lists show only their first entries)

```json
{
  "attribution": "mapx.at; Fahrplandaten: OeBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)",
  "departures": [
    {
      "destination": {
        "id": "at:43:7227",
        "name": "Pöchlarn Bahnhof"
      },
      "headsign": "Pöchlarn Bahnhof",
      "line": {
        "long_name": "Wien - Salzburg - Kufstein - Innsbruck - Bludenz - Bregenz",
        "mode": "rail",
        "name": "R52",
        "route_type": 2,
        "short_name": "R52",
        "trip_short_name": "R 2004"
      },
      "operator": "OEBB Personenverkehr AG Kundenservice",
      "platform": "2",
      "realtime": false,
      "scheduled": "2026-11-23T08:00:00+01:00",
      "stop": {
        "id": "at:43:7226",
        "name": "Melk Bahnhof"
      },
      "time": "2026-11-23T08:00:00+01:00",
      "trip_id": "oebb:28.TA.3-R52-A-j26-1.9.H"
    }
  ],
  "realtime": false,
  "stops": [
    {
      "id": "at:43:7226",
      "lines": [
        "671",
        "683"
      ],
      "location": [
        15.331017,
        48.224588
      ],
      "modes": [
        "bus",
        "rail"
      ],
      "name": "Melk Bahnhof"
    }
  ],
  "time": "2026-11-23T08:00:00+01:00"
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

## Stops (GTFS stations merged by IFOPT)

`GET /v1/stops`

Stations in a box (`bbox`, by number of trip patterns descending) or around a point (`near`, nearest first, with `distance` in meters). Per station `id`, `name`, `location` [lon,lat], `modes` (rail, bus, tram, subway, ...) and `lines`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north in degrees, at most 1 square degree |
| `near` | query | string |  | – | lon,lat, instead of bbox |
| `radius` | query | string |  | `500` | With near only: meters, 1 to 5000, default 500 |
| `limit` | query | string |  | `50` | 1 to 500, default 50 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response** (shortened: lists show only their first entries)

```json
{
  "attribution": "mapx.at; Fahrplandaten: OeBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)",
  "stops": [
    {
      "distance": 219,
      "id": "at:43:7226",
      "lines": [
        "671",
        "683"
      ],
      "location": [
        15.331017,
        48.224588
      ],
      "modes": [
        "bus",
        "rail"
      ],
      "name": "Melk Bahnhof"
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
| `502` | Upstream service unavailable |

## Playground

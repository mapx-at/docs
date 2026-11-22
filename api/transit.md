# Transit API: public transport Austria

Source: https://docs.mapx.at/api/transit/

> Transit API for public transport in Austria: journeys, stops and departure boards from the timetable data, with parameters and examples.

Journeys with public transport (on foot or by bike to the stop), departure boards and stops, based on the scheduled timetable.

Base URL: `https://api.mapx.at` · Rate-limit zone: `route` (5/s), `search` (10/s) · Guide: [Transit departures](https://docs.mapx.at/guides/transit.md)

## Public transit: walking plus transit (GTFS timetable, RAPTOR)

`GET /v1/directions/transit/{coordinates}`

Exactly origin and destination. Connections with walks on the GIP.at network and rides (line, stops, times, `platform` at `from`, `to` and `stops[]` where known, otherwise null). Time via `depart_at` (default now) or `arrive_by`, ISO 8601 in Vienna local time; invalid dates or times such as `T25:00` or `2026-02-31` answer 400.

`line.name` is the public line name as in `/v1/departures` (`U3`, `S50`, `R52`, long-distance trains by train number such as `RJ 754`), `trip_short_name` the train number. Rides carry the mode of transport in `mode` and `line.mode` (rail, subway, tram, bus, ferry, cable, other; the same values as in `/v1/stops` and `/v1/departures`), walks `walking` or `cycling`. `modes` restricts the rides to these modes (walks stay). Changing at the same station takes at least 2 minutes, after a walk at least 1 minute remains before the departure. At most `routes` connections, sorted by arrival (`depart_at`) or by departure, latest first (`arrive_by`); connections that are no better than another (not earlier, not faster, not fewer rides) and coupled trains with identical times appear once. A direct walk comes first when it is not slower than the best ride. `timetable` gives the loaded timetable period (`valid_from`, `valid_to`); outside it, and when no stop is within reach or no connection is found, 422. Timetable data: ÖBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitätsverbünde Österreich OG, data.mobilitaetsverbuende.at.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | path | string | yes | – | lng,lat;lng,lat (exactly origin and destination) |
| `depart_at` | query | string |  | – | Earliest departure, e.g. 2026-11-09T08:00 |
| `arrive_by` | query | string |  | – | Latest arrival, instead of depart_at |
| `routes` | query | string |  | `3` | 1 to 5, default 3 |
| `modes` | query | string |  | `all` | Only rides with these modes of transport, comma-separated: rail (long-distance, regional and S-Bahn trains), subway, tram, bus (incl. trolleybus), ferry, cable (cable car, funicular, gondola), other; default all. Walks and the bike leg stay |

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
  "attribution": "mapx.at; ÖBB-Personenverkehr AG (CC BY 4.0); Mobilitätsverbünde Österreich OG (data.mobilitaetsverbuende.at); GIP.at (CC BY 4.0)",
  "query": {
    "depart_at": "2026-11-23T08:00:00+01:00",
    "profile": "transit"
  },
  "routes": [
    {
      "arrival": "2026-11-23T08:15:51+01:00",
      "departure": "2026-11-23T08:01:26+01:00",
      "distance": 4801,
      "duration": 865,
      "geometry": {
        "coordinates": [
          [
            16.3725,
            48.2085
          ]
        ],
        "type": "LineString"
      },
      "legs": [
        {
          "arrival": "2026-11-23T08:03:00+01:00",
          "departure": "2026-11-23T08:01:26+01:00",
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
      "weight": 865,
      "weight_name": "duration"
    }
  ],
  "timetable": {
    "valid_from": "2025-12-14",
    "valid_to": "2027-12-11"
  }
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Bike to the stop plus transit

`GET /v1/directions/bike_transit/{coordinates}`

Like transit, but the first leg by bike (mode `cycling`, up to 15 min). The bike stays at the boarding stop (bikes are not taken on board), changes and the last leg are walked. Short trips may come as a direct bike route.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | path | string | yes | – | lng,lat;lng,lat (exactly origin and destination) |
| `depart_at` | query | string |  | – | Earliest departure |
| `arrive_by` | query | string |  | – | Latest arrival |
| `routes` | query | string |  | `3` | 1 to 5, default 3 |
| `modes` | query | string |  | `all` | Only rides with these modes of transport, comma-separated: rail (long-distance, regional and S-Bahn trains), subway, tram, bus (incl. trolleybus), ferry, cable (cable car, funicular, gondola), other; default all. Walks and the bike leg stay |

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
  "attribution": "mapx.at; ÖBB-Personenverkehr AG (CC BY 4.0); Mobilitätsverbünde Österreich OG (data.mobilitaetsverbuende.at); GIP.at (CC BY 4.0)",
  "query": {
    "depart_at": "2026-11-23T08:00:00+01:00",
    "profile": "transit"
  },
  "routes": [
    {
      "arrival": "2026-11-23T08:15:51+01:00",
      "departure": "2026-11-23T08:01:26+01:00",
      "distance": 4801,
      "duration": 865,
      "geometry": {
        "coordinates": [
          [
            16.3725,
            48.2085
          ]
        ],
        "type": "LineString"
      },
      "legs": [
        {
          "arrival": "2026-11-23T08:03:00+01:00",
          "departure": "2026-11-23T08:01:26+01:00",
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
      "weight": 865,
      "weight_name": "duration"
    }
  ],
  "timetable": {
    "valid_from": "2025-12-14",
    "valid_to": "2027-12-11"
  }
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Departure board (scheduled GTFS timetable)

`GET /v1/departures`

Next departures at a station (id from `/v1/stops`, e.g. `at:43:7226`) or at all stations around a coordinate. Service days follow calendar and feed period, trips past midnight included; times are exact on the days clocks change (GTFS times count from noon minus 12 h).

Per departure: time, line (`name` as in transit: `U1`, `R52`, `RJ 754`; plus `short_name`, `trip_short_name`, `mode`, `route_type`), headsign, destination, platform (null if unknown), operator, stop. Times come from the scheduled timetable: `realtime` is always false. `timetable` gives the loaded timetable period (`valid_from`, `valid_to`, all timetable years combined). Outside it 422, unknown station 404, invalid time 400. Timetable data: ÖBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitätsverbünde Österreich OG, data.mobilitaetsverbuende.at.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `stop` | query | string | yes | – | Station id (IFOPT `at:<state>:<nr>`, all platforms), platform id such as `at:43:7226:0:2` (only this platform; `stops[0]` then carries `platform_id` and `platform`) or lng,lat |
| `time` | query | string |  | `now` | Start time, ISO 8601 (Vienna local time, or with `Z`/offset) or Unix seconds, default now; a local time in the spring-forward gap moves on to summer time |
| `limit` | query | string |  | `20` | 1 to 100, default 20 |
| `radius` | query | string |  | `300` | With lng,lat only: meters, 1 to 2000, default 300 |
| `horizon` | query | string |  | `14400` | Window in seconds, 60 to 86400, default 14400 |
| `modes` | query | string |  | `all` | Only departures of these modes of transport, comma-separated: rail (long-distance, regional and S-Bahn trains), subway, tram, bus (incl. trolleybus), ferry, cable (cable car, funicular, gondola), other; default all. With lng,lat only stations served by one of them; `stops[]` lists only these modes and lines |

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
  "attribution": "mapx.at; ÖBB-Personenverkehr AG (CC BY 4.0); Mobilitätsverbünde Österreich OG (data.mobilitaetsverbuende.at); GIP.at (CC BY 4.0)",
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
  "time": "2026-11-23T08:00:00+01:00",
  "timetable": {
    "valid_from": "2025-12-14",
    "valid_to": "2027-12-11"
  }
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

## Stops (GTFS stations merged by IFOPT)

`GET /v1/stops`

Stations in a box (`bbox`, by number of trip patterns descending) or around a point (`near`, nearest first, with `distance` in meters). Per station `id`, `name`, `location` [lng,lat], `modes` (rail, subway, tram, bus, ferry, cable, other) and `lines` (public line names as in `/v1/departures`, long-distance trains by category such as `RJ`, `RJX`, `IC`).

With `modes` only stations served by one of these modes, and `modes`/`lines` list only them. Coordinates given as lat,lng answer 400.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north in degrees, at most 1 square degree |
| `near` | query | string |  | – | lng,lat, instead of bbox |
| `radius` | query | string |  | `500` | With near only: meters, 1 to 5000, default 500 |
| `limit` | query | string |  | `50` | 1 to 500, default 50 |
| `modes` | query | string |  | `all` | Only stations served by these modes of transport, comma-separated: rail (long-distance, regional and S-Bahn trains), subway, tram, bus (incl. trolleybus), ferry, cable (cable car, funicular, gondola), other; default all |

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
  "attribution": "mapx.at; ÖBB-Personenverkehr AG (CC BY 4.0); Mobilitätsverbünde Österreich OG (data.mobilitaetsverbuende.at); GIP.at (CC BY 4.0)",
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
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

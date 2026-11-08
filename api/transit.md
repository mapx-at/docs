# Transit API

Public transport journeys, departure boards and stops from the timetable data.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Journeys with public transport (on foot or by bike to the stop), departure boards and stops, based on the scheduled timetable.

Base URL: `https://api.mapx.at` · Rate-limit zone: `route` (5/s), `search` (10/s) · Guide: [Transit departures](/guides/transit/)

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

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "code": "Ok",
  "routes": [
    {
      "duration": 2400,
      "legs": [
        {
          "mode": "walking",
          "duration": 300
        },
        {
          "mode": "transit",
          "line": "S1",
          "from": "Wien Hbf",
          "to": "Wien Floridsdorf"
        }
      ]
    }
  ],
  "waypoints": []
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Departure board (scheduled GTFS timetable)

`GET /v1/departures`

Next departures at a station (id from `/v1/stops`, e.g. `at:43:7226`) or at all stations around a coordinate. Service days follow calendar and feed period, trips past midnight included. Per departure: time, line (name, mode, route_type), headsign, destination, platform (null if unknown), operator, stop.

<details>
<summary>More</summary>

No real-time data: `realtime` is always false. Outside the feed period 422, unknown station 404. Timetable data: OeBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitaetsverbuende Oesterreich OG, data.mobilitaetsverbuende.at.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `stop` | query | string | yes | – | Station id (IFOPT `at:<state>:<nr>`, also platform id `at:43:7226:0:2`) or lon,lat |
| `time` | query | string |  | `now` | Start time, ISO 8601 (Vienna local) or Unix seconds, default now |
| `limit` | query | string |  | `20` | 1 to 100, default 20 |
| `radius` | query | string |  | `300` | With lon,lat only: meters, 1 to 2000, default 300 |
| `horizon` | query | string |  | `14400` | Window in seconds, 60 to 86400, default 14400 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "time": "2026-11-09T08:00:00+01:00",
  "realtime": false,
  "stops": [
    {
      "id": "at:43:7226",
      "name": "Melk Bahnhof"
    }
  ],
  "departures": [
    {
      "time": "2026-11-09T08:04:00+01:00",
      "line": {
        "name": "REX 1",
        "mode": "rail",
        "route_type": 2
      },
      "headsign": "Wien Westbahnhof",
      "platform": "2",
      "stop": "at:43:7226"
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
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

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "stops": [
    {
      "id": "at:43:7226",
      "name": "Melk Bahnhof",
      "location": [
        15.3338,
        48.2279
      ],
      "modes": [
        "rail",
        "bus"
      ],
      "lines": [
        "REX 1",
        "S50"
      ]
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

<Playground group="transit" lang="en" />
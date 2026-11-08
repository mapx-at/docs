# Transit API

ÖV-Verbindungen, Abfahrtstafeln und Haltestellen aus den Fahrplandaten.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Verbindungen mit öffentlichem Verkehr (zu Fuß oder mit dem Rad zur Haltestelle), Abfahrtstafeln und Haltestellen auf Basis des Fahrplans.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `route` (5/s), `search` (10/s) · Guide: [ÖV-Abfahrten](/de/guides/transit/)

## Public transit: walking plus transit (GTFS timetable, RAPTOR)

`GET /v1/directions/transit/{coordinates}`

Exactly origin and destination. Connections with walks on the GIP.at network and rides (line, stops, times). Time via `depart_at` (default now) or `arrive_by`, ISO 8601 in Vienna local time. Timetable data: OeBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitaetsverbuende Oesterreich OG, data.mobilitaetsverbuende.at.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | path | string | ja | – | lon,lat;lon,lat (exactly origin and destination) |
| `depart_at` | query | string |  | – | Earliest departure, e.g. 2026-11-09T08:00 |
| `arrive_by` | query | string |  | – | Latest arrival, instead of depart_at |
| `routes` | query | string |  | `3` | 1 to 5, default 3 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00")
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Departure board (scheduled GTFS timetable)

`GET /v1/departures`

Next departures at a station (id from `/v1/stops`, e.g. `at:43:7226`) or at all stations around a coordinate. Service days follow calendar and feed period, trips past midnight included. Per departure: time, line (name, mode, route_type), headsign, destination, platform (null if unknown), operator, stop.

<details>
<summary>Mehr</summary>

No real-time data: `realtime` is always false. Outside the feed period 422, unknown station 404. Timetable data: OeBB-Personenverkehr AG (CC BY 4.0); data source: Mobilitaetsverbuende Oesterreich OG, data.mobilitaetsverbuende.at.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `stop` | query | string | ja | – | Station id (IFOPT `at:<state>:<nr>`, also platform id `at:43:7226:0:2`) or lon,lat |
| `time` | query | string |  | `now` | Start time, ISO 8601 (Vienna local) or Unix seconds, default now |
| `limit` | query | string |  | `20` | 1 to 100, default 20 |
| `radius` | query | string |  | `300` | With lon,lat only: meters, 1 to 2000, default 300 |
| `horizon` | query | string |  | `14400` | Window in seconds, 60 to 86400, default 14400 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300")
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Stops (GTFS stations merged by IFOPT)

`GET /v1/stops`

Stations in a box (`bbox`, by number of trip patterns descending) or around a point (`near`, nearest first, with `distance` in meters). Per station `id`, `name`, `location` [lon,lat], `modes` (rail, bus, tram, subway, ...) and `lines`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north in degrees, at most 1 square degree |
| `near` | query | string |  | – | lon,lat, instead of bbox |
| `radius` | query | string |  | `500` | With near only: meters, 1 to 5000, default 500 |
| `limit` | query | string |  | `50` | 1 to 500, default 50 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5")
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

<Playground group="transit" lang="de" />
# Public transport

Journey planning with a walk or bike leg to the stop, departure board and stops from the open GTFS timetables of ÖBB and the transport associations.

The planner combines the timetable (GTFS, RAPTOR routing) with the walking and cycling network [GIP.at](https://www.gip.at). Access legs, transfer walks and egress are computed on the real path network. Timetable data: ÖBB-Personenverkehr AG, [scheduled timetable](https://data.oebb.at/de/datensaetze~soll-fahrplan-gtfs~) under CC BY 4.0. In addition, the timetables of the transport associations (VOR, Verbundlinie, Salzburg, Carinthia, OÖVV, VVT, Vmobil, Linz AG) from [data.mobilitaetsverbuende.at](https://data.mobilitaetsverbuende.at) under the Mobilitätsverbünde Österreich data licence: city, regional and rural buses, trams and underground. The 2026 timetable is valid from 14 Dec 2025 to 12 Dec 2026; other days return `422`.

## Journeys

**GET https://api.mapx.at/v1/directions/transit/{coordinates}**

Bike to the stop: `GET /v1/directions/bike_transit/{coordinates}`.

<ParamTable params={[
  { name: "coordinates", type: "string", required: true, desc: "Exactly origin and destination as <code>lon,lat;lon,lat</code>." },
  { name: "depart_at", type: "string", desc: "Earliest departure, ISO 8601 in Vienna local time, e.g. <code>2026-11-09T08:00</code>. An offset (<code>Z</code>, <code>+01:00</code>) is accepted. Default: now." },
  { name: "arrive_by", type: "string", desc: "Latest arrival, instead of <code>depart_at</code>. The search runs backwards; the first journey leaves as late as possible." },
  { name: "routes", type: "integer", default: "3", desc: "Number of journeys, 1 to 5." },
]} />

Profiles: `transit` (walk up to 20 minutes to the stop) and `bike_transit` (cycle up to 15 minutes; bikes are not taken on vehicles). Transfers between stops are always walks of up to 8 minutes; at the same station at least 2 minutes apply.

Response: `code`, `routes[]`, `waypoints[]`, `query` and `attribution`. Each route has `duration` (seconds), `distance` (metres), `departure`, `arrival` (ISO 8601), `transfers`, `summary`, `geometry` (GeoJSON) and `legs[]`. Each leg has a `mode`:

- `walking` or `cycling`: `duration`, `distance`, `from`, `to`, `geometry`. The first leg ends at the departure, the last one starts at the arrival.
- `transit`: `line` (`name` such as "RJX 61", `long_name`, `type` such as `rail` or `bus`, `agency`, `headsign`), `from`, `to`, `stops[]` (with arrival and departure per stop) and `geometry` as a line through the stops. The line follows the stops, not the track.

If the destination is close, `routes[]` also contains the plain walk (or ride).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/transit/15.3338,48.2279;16.3770,48.1850?depart_at=2026-11-09T08:00"`} />

## Departure board

**GET https://api.mapx.at/v1/departures**

<ParamTable params={[
  { name: "stop", type: "string", required: true, desc: "Station id from <code>/v1/stops</code> or search (<code>stop_id</code>), e.g. <code>at:43:7226</code> (IFOPT, a platform id like <code>at:43:7226:0:2</code> works too), or <code>lon,lat</code>: all stations within <code>radius</code>." },
  { name: "time", type: "string", desc: "From when, ISO 8601 in Vienna local time or Unix seconds. Default: now." },
  { name: "limit", type: "integer", default: "20", desc: "Number of departures, 1 to 100." },
  { name: "radius", type: "integer", default: "300", desc: "Only with <code>lon,lat</code>: radius in metres, 1 to 2000." },
  { name: "horizon", type: "integer", default: "14400", desc: "Time window in seconds after <code>time</code>, 60 to 86400." },
]} />

Next departures from the **scheduled timetable**: service days follow the calendar and timetable period (including exception days and trips that run past midnight from the previous day). There is no real-time data, `realtime` is always `false`. A trip has no departure at its final stop.

Response: `time`, `realtime`, `stops[]` (the selected stations) and `departures[]`, each with `time`, `scheduled`, `realtime`, `line` (`name`, `short_name`, `long_name`, `trip_short_name` such as "R 2004", `mode` such as `rail`, `bus`, `tram`, `subway`, `route_type`), `headsign`, `destination` (`id`, `name`), `platform` (`null` if unknown), `operator`, `stop` and `trip_id`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/departures?stop=at:43:7226&time=2026-11-09T08:00&limit=5"`} />

## Stops

**GET https://api.mapx.at/v1/stops**

<ParamTable params={[
  { name: "bbox", type: "string", desc: "<code>west,south,east,north</code>, at most 1 square degree. Largest hubs first." },
  { name: "near", type: "string", desc: "<code>lon,lat</code>, instead of <code>bbox</code>: nearest first, with <code>distance</code> in metres." },
  { name: "radius", type: "integer", default: "500", desc: "Only with <code>near</code>: metres, 1 to 5000." },
  { name: "limit", type: "integer", default: "50", desc: "1 to 500." },
]} />

Stations from all feeds; platforms and entries of different transport associations with the same IFOPT id (`at:<state>:<no>`) are merged into one station. Each station has `id`, `name`, `location` (`[lon, lat]`), `modes` and `lines`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/stops?near=16.3752,48.1851&radius=300"`} />

## Errors

| Status | Meaning |
|---|---|
| 400 | invalid coordinates or time, <code>depart_at</code> and <code>arrive_by</code> together |
| 404 | no stop within reach (or unknown <code>stop</code>), or no journey within 48 hours |
| 422 | date outside the timetable period |

Journeys are not cached, departures for 30 seconds, stops for one hour. Credit: "Timetable: ÖBB-Personenverkehr AG (CC BY 4.0), network: GIP.at (CC BY 4.0)".
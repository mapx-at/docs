# Public transport

Journey planning with a walk or bike leg to the stop and a timetable from the open ÖBB GTFS data.

The planner combines the timetable (GTFS, RAPTOR routing) with the walking and cycling network [GIP.at](https://www.gip.at). Access legs, transfer walks and egress are computed on the real path network. Timetable data: ÖBB-Personenverkehr AG, [scheduled timetable](https://data.oebb.at/de/datensaetze~soll-fahrplan-gtfs~) under CC BY 4.0. The data covers the **rail network and ÖBB buses**; city transport (Wiener Linien, trams, regional buses of the transport associations) is not included yet. The 2026 timetable is valid from 14 Dec 2025 to 12 Dec 2026; other days return `422`.

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

## Errors

| Status | Meaning |
|---|---|
| 400 | invalid coordinates or time, <code>depart_at</code> and <code>arrive_by</code> together |
| 404 | no stop within reach, or no journey within 48 hours |
| 422 | date outside the timetable period |

Responses are not cached. Credit: "Timetable: ÖBB-Personenverkehr AG (CC BY 4.0), network: GIP.at (CC BY 4.0)".
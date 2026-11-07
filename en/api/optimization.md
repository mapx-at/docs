# Vehicle routing

Tours for several vehicles with capacity, time windows and pickup/delivery jobs, simplified after Mapbox Optimization v2.

Distributes jobs across vehicles and puts them into a good order for each vehicle. Travel times come from the matrix on the [GIP.at](https://www.gip.at) network. For a single trip in the best order, [Optimized trips](/en/api/routing/#optimized-trips) is enough.

**POST https://api.mapx.at/v1/optimization**

The body is JSON (`Content-Type: application/json`, at most 256 KB). Times are seconds, quantities integers (several dimensions are possible, such as weight and volume), coordinates `[lon, lat]`.

## Input

<ParamTable params={[
  { name: "profile", type: "string", default: "driving", desc: "<code>driving</code>, <code>cycling</code> or <code>walking</code>." },
  { name: "vehicles", type: "array", required: true, desc: "1 to 10 vehicles: <code>id</code>, <code>start</code> (required), <code>end</code> (without it the tour ends at the last job), <code>capacity</code> (list, unlimited if omitted), <code>time_window</code> (<code>[from, to]</code>, default unlimited from 0)." },
  { name: "services", type: "array", desc: "Jobs at one place: <code>id</code>, <code>location</code>, <code>service</code> (duration, default 0), <code>delivery</code> (quantity loaded at the start and dropped here, also <code>amount</code>), <code>pickup</code> (quantity picked up here), <code>time_windows</code> (list of <code>[from, to]</code>)." },
  { name: "shipments", type: "array", desc: "Pickup-and-delivery jobs: <code>id</code>, <code>amount</code>, <code>pickup</code> and <code>delivery</code> each with <code>location</code>, <code>service</code>, <code>time_windows</code>. Pickup comes before delivery, both in the same vehicle." },
  { name: "options.time_limit_ms", type: "integer", default: "1500", desc: "Search time, 50 to 10000." },
  { name: "options.geometry", type: "boolean", default: "false", desc: "With <code>true</code> every tour gets a <code>geometry</code> (at most 80 legs in total); format via <code>options.geometries</code>." },
]} />

Limits: 10 vehicles, 50 jobs (services plus shipments), 60 distinct places. Ids must be unique.

## Example

<Code lang="bash" code={`curl -X POST https://api.mapx.at/v1/optimization -H 'Content-Type: application/json' -d '{   "profile": "driving",   "vehicles": [     { "id": "melk", "start": [15.3338, 48.2279], "end": [15.3338, 48.2279], "capacity": [3], "time_window": [28800, 64800] },     { "id": "ybbs", "start": [15.0864, 48.1696], "capacity": [3], "time_window": [28800, 64800] }   ],   "services": [     { "id": "poechlarn", "location": [15.2065, 48.2111], "service": 600, "delivery": [1] },     { "id": "loosdorf", "location": [15.3969, 48.2339], "service": 600, "delivery": [1], "time_windows": [[28800, 36000]] },     { "id": "mank", "location": [15.3167, 48.1167], "service": 900, "delivery": [2] }   ],   "shipments": [     { "id": "paket", "amount": [1],       "pickup": { "location": [15.2597, 48.2018], "service": 120 },       "delivery": { "location": [15.0764, 48.1692], "service": 120 } }   ],   "options": { "time_limit_ms": 1500, "geometry": true } }'`} />

Request (readable):

<Code lang="json" code={`{
  "profile": "driving",
  "vehicles": [
    { "id": "melk", "start": [15.3338, 48.2279], "end": [15.3338, 48.2279], "capacity": [3], "time_window": [28800, 64800] },
    { "id": "ybbs", "start": [15.0864, 48.1696], "capacity": [3], "time_window": [28800, 64800] }
  ],
  "services": [
    { "id": "poechlarn", "location": [15.2065, 48.2111], "service": 600, "delivery": [1] },
    { "id": "loosdorf", "location": [15.3969, 48.2339], "service": 600, "delivery": [1], "time_windows": [[28800, 36000]] },
    { "id": "mank", "location": [15.3167, 48.1167], "service": 900, "delivery": [2] }
  ],
  "shipments": [
    { "id": "paket", "amount": [1],
      "pickup": { "location": [15.2597, 48.2018], "service": 120 },
      "delivery": { "location": [15.0764, 48.1692], "service": 120 } }
  ],
  "options": { "time_limit_ms": 1500, "geometry": true }
}`} />

## Response

<Code lang="json" code={`{
  "code": "Ok",
  "summary": { "cost": 5563.6, "duration": 5563.6, "distance": 104708.3, "service": 3240.0, "waiting_time": 0.0, "routes": 2, "unassigned": 0, "computing_ms": 1517 },
  "routes": [
    { "vehicle": "melk", "duration": 4080.0, "distance": 76800.0, "end_time": 34560.0,
      "steps": [
        { "type": "start", "location": [15.3338, 48.2279], "arrival": 28800.0, "load": [3] },
        { "type": "job", "id": "loosdorf", "location": [15.3969, 48.2339], "arrival": 29280.0, "waiting_time": 0.0, "service": 600.0, "load": [2] },
        { "type": "end", "location": [15.3338, 48.2279], "arrival": 34560.0, "load": [0] }
      ] }
  ],
  "unassigned": []
}`} />

- `summary`: totals over all tours (`cost` = travel time in seconds, `distance` in metres, `service`, `waiting_time`), number of tours and unassigned jobs, `computing_ms`.
- `routes[]`: one tour per used vehicle. `steps[]` has the types `start`, `job`, `pickup`, `delivery`, `end`, plus `arrival` (seconds), `waiting_time`, `service` and `load` after the stop.
- `unassigned[]`: jobs without a place, with `reason` (capacity, reachability, time windows).

## Method and limits

The goal is first as many jobs as possible, then the smallest total travel time. The search uses regret insertion, local search (relocate, reverse) and ruin-and-recreate until the time limit; the result is good but not provably optimal. Vehicles start at the beginning of their time window and wait before a time window if needed; a later start that avoids waiting is not searched. There are no skills, priorities, breaks or fixed vehicle costs.

The request counts as expensive and has a tight limit. Errors: `400` invalid input (naming the field), `429` rate limit, `502` routing unreachable. General: [Errors and limits](/en/api/fehler/).
# Transit departures

Public transport journeys, stops and departure boards from the scheduled timetable.

The timetable data covers Austria (ÖBB and the regional transport associations). It is the scheduled timetable: `realtime` is always `false`. Requests are in the `route` zone for journeys and the `search` zone for stops and departures.

| Task | Endpoint |
|---|---|
| Find stops near a point | [`/v1/stops`](/api/transit/) |
| Next departures at a stop | [`/v1/departures`](/api/transit/) |
| Journey from A to B | [`/v1/directions/transit/{coordinates}`](/api/transit/) |

## Find stops

```bash
curl -s "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq '.stops[] | {id, name, modes, lines}'
```

Stops of the same station from different operators are merged and share one `id` (for example `at:49:1320`). Use `bbox` (at most one square degree) to list a map section.

## Departure board

Pass a stop `id`, or a coordinate to get the departures of all stops around it.

```js
const url = "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=8";
const { departures } = await (await fetch(url)).json();
for (const d of departures) console.log(d.time.slice(11, 16), d.line.name, d.headsign, d.platform ?? "");
```

`time` is ISO 8601 in Vienna local time. Use `time=2026-11-09T08:00` for a later start and `horizon` (seconds, up to 24 hours) for the window. Outside the timetable period the API answers `422`.

## Journey planning

```bash
curl -s "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00&routes=3" | jq '.routes[0].duration'
```

Pass exactly an origin and a destination. Use `depart_at` (earliest departure) or `arrive_by` (latest arrival), default is now. The response contains walking sections on the road network and rides with line, stops and times.

## Reachability by transit

Walking plus public transport as isochrones: profile `transit` of the isochrone API. The search walks to the stops around the start, rides the scheduled timetable from `depart_at` and walks on from every stop reached.

```bash
curl -s "https://api.mapx.at/v1/isochrone/transit/16.3725,48.2085?contours_minutes=10,20,30&depart_at=2026-11-10T08:00" | jq '.features[] | {contour: .properties.contour, km2: .properties.area_km2}'
```

Without `depart_at` the isochrone starts now (Vienna local time). Only `contours_minutes` applies. The area depends strongly on the time: the same start has a much smaller area at night. Outside the timetable period the API answers `422`.

## Show departures in an app

1. Search a stop with [`/v1/stops`](/api/transit/) or let the user click the map.
2. Call `/v1/departures` and refresh every minute.
3. Show line, destination and `time`; mark missing real-time data with "scheduled".

## Next

- [Transit API](/api/transit/)
- [Navigation](/guides/navigation/)
- [Attribution](/reference/attribution/): timetable data credit
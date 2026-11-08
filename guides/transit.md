# Transit departures

The timetable data covers Austria (ÖBB and the regional transport associations). It is the scheduled timetable: `realtime` is always `false`. Requests are in the `route` zone for journeys and the `search` zone for stops and departures. Every request needs a token ([Access and limits](/start/access-and-limits/)); `curl` samples send it as a header, browser samples as `access_token`.

| Task | Endpoint |
|---|---|
| Find stops near a point | [`/v1/stops`](/api/transit/) |
| Next departures at a stop | [`/v1/departures`](/api/transit/) |
| Journey from A to B | [`/v1/directions/transit/{coordinates}`](/api/transit/) |

## Find stops

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq '.stops[] | {id, name, modes, lines}'
```

Stops of the same station from different operators are merged and share one `id` (for example `at:49:1320`). Use `bbox` (at most one square degree) to list a map section.

## Departure board

Pass a stop `id`, or a coordinate to get the departures of all stops around it.

```js
const url = "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=8&access_token=YOUR_TOKEN";
const { departures } = await (await fetch(url)).json();
for (const d of departures) console.log(d.time.slice(11, 16), d.line.name, d.headsign, d.platform ?? "");
```

`time` is ISO 8601 in Vienna local time. Use `time=2026-11-09T08:00` for a later start and `horizon` (seconds, up to 24 hours) for the window. Outside the timetable period the API answers `422`.

## Journey planning

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00&routes=3" | jq '.routes[0].duration'
```

Pass exactly an origin and a destination. Use `depart_at` (earliest departure) or `arrive_by` (latest arrival), default is now. The response contains walking sections on the road network and rides with line, stops and times.

## Reachability by transit

Walking plus public transport as isochrones: profile `transit` of the isochrone API. The search walks to the stops around the start, rides the scheduled timetable from `depart_at` and walks on from every stop reached.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/transit/16.3725,48.2085?contours_minutes=10,20,30&depart_at=2026-11-10T08:00" | jq '.features[] | {contour: .properties.contour, km2: .properties.area_km2}'
```

Without `depart_at` the isochrone starts now (Vienna local time). Only `contours_minutes` applies. The area depends strongly on the time: the same start has a much smaller area at night. Outside the timetable period the API answers `422`.

### Access by bike or car (Bike & Ride, Park & Ride)

With `access=bike` or `access=car` the first leg is not walked but cycled or driven to a stop. Bikes can be locked at **every** stop (+2 min); cars park at stops with rail, metro or tram service (+3 min, at most 25 min drive); the algorithm takes the fastest combination, so the station in your own village wins over driving on to the next town if the train is quicker. Official P+R sites from the GIP are added for the car (+2 min, then a walk to the stop). Walking from the start still applies, the minimum counts. The exit is always on foot.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/transit/15.16,48.207?contours_minutes=45&depart_at=2026-11-09T07:00&access=car" | jq '.metadata | {access, access_stops_improved, sites: .access_sites[:3]}'
```

| Parameter | Values | Meaning |
|---|---|---|
| `access` | `foot` (default), `bike`, `car` | Mode to the first stop |
| `park` | `any` (default), `official` | `official`: bike only at stops within 200 m of a B+R site, car only at official P+R sites |
| `sites` | 1 to 50 (default 10) | Entries in `metadata.access_sites` |

`metadata.access_sites` lists the accesses that beat walking, sorted by travel time: `source` (`site` for an official P+R, `stop` for parking at the stop), `kind`, `name`, `lon`, `lat`, `arrive_min` (travel time), `ready_min` (ready to board, incl. parking and walk) and `stops_won`. Bringing the bike on the train (`egress=bike`) is not offered: the timetable data carry no bike-carriage information.

## Show departures in an app

1. Search a stop with [`/v1/stops`](/api/transit/) or let the user click the map.
2. Call `/v1/departures` and refresh every minute.
3. Show line, destination and `time`; mark missing real-time data with "scheduled".

## Next

- [Transit API](/api/transit/)
- [Navigation](/guides/navigation/)
- [Attribution](/reference/attribution/): timetable data credit


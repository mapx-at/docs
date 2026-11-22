# Transit guide: public transport departures

Source: https://docs.mapx.at/guides/transit/

> Query public transport journeys, stops and departure boards in Austria from the scheduled timetable with the mapx transit API.

The timetable data covers Austria (ÖBB and the regional transport associations). It is the scheduled timetable: `realtime` is always `false`. Requests are in the `route` zone for journeys and the `search` zone for stops and departures. Every request needs a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); the samples send it in the `Authorization` header, which browsers may send too.

| Task | Endpoint |
|---|---|
| Find stops near a point | [`/v1/stops`](https://docs.mapx.at/api/transit.md) |
| Next departures at a stop | [`/v1/departures`](https://docs.mapx.at/api/transit.md) |
| Journey from A to B | [`/v1/directions/transit/{coordinates}`](https://docs.mapx.at/api/transit.md) |

## Find stops

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq '.stops[] | {id, name, modes, lines}'
```

Stops of the same station from different operators are merged and share one `id` (for example `at:49:1320`). Use `bbox` (at most one square degree) to list a map section.

## Departure board

Pass a stop `id`, or a coordinate to get the departures of all stops around it. A platform id such as `at:43:7226:0:2` returns only the departures from that platform (`stops[0]` then carries `platform_id` and `platform`); the station id returns all platforms.

```js
const url = "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=8";
const { departures } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
for (const d of departures) console.log(d.time.slice(11, 16), d.line.name, d.headsign, d.platform ?? "");
```

`time` is ISO 8601 in Vienna local time. Without `time` the request starts now. Use `time` (format `YYYY-MM-DDThh:mm`, Vienna local time) for a later start and `horizon` (seconds, up to 24 hours) for the window. The timetable is valid for a limited period only: `timetable.valid_from` and `timetable.valid_to` in every response name it (around the December timetable change both timetable years are loaded together). Outside it the API answers `422`, and the message (`detail`) names the valid period. Invalid dates or times such as `2026-02-31` or `T25:00` answer `400`. On the days the clocks change the times are exact; a local time that does not exist (between 02:00 and 03:00 in spring) moves on to summer time.

`line.name` is the line name passengers see, the same in departures and journeys: `U1`, `S50`, `R52`, bus numbers, long-distance trains by train number such as `RJ 754`. `trip_short_name` holds the train number.

## Journey planning

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855" | jq '.routes[0].duration'
```

Pass exactly an origin and a destination. Use `depart_at` (earliest departure) or `arrive_by` (latest arrival), each in the format `YYYY-MM-DDThh:mm` in Vienna local time; the default is now. The response contains several journeys in `routes[]` (the number can be set with `routes`, see the [transit API](https://docs.mapx.at/api/transit.md)), with walking sections on the road network and rides with line, stops, times and platform (`from.platform`, `to.platform`, `stops[].platform`, `null` if unknown). Changing at the same station takes at least 2 minutes, after a walk at least 1 minute remains before the departure. With `bike_transit` the first leg is cycled; the bike stays at the stop, changes and the last leg are walked. No stop within reach or no connection: `422`.

## Reachability by transit

Walking plus public transport as isochrones: profile `transit` of the isochrone API. The search walks to the stops around the start, rides the scheduled timetable from `depart_at` and walks on from every stop reached.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/transit/16.3725,48.2085?contours_minutes=10,20,30" | jq '.features[] | {contour: .properties.contour, km2: .properties.area_km2}'
```

Without `depart_at` the isochrone starts now (Vienna local time); use `depart_at` to pick another time. Only `contours_minutes` applies. The area depends strongly on the time: the same start has a much smaller area at night. Outside the timetable period the API answers `422`.

### Access by bike or car (Bike & Ride, Park & Ride)

With `access=bike` or `access=car` the first leg is not walked but cycled or driven to a stop. Bikes can be locked at **every** stop (+2 min); cars park at stops with rail, metro or tram service (+3 min, at most 25 min drive); the algorithm takes the fastest combination, so the station in your own village wins over driving on to the next town if the train is quicker. Official P+R sites from the GIP are added for the car (+2 min, then a walk to the stop). Walking from the start still applies, the minimum counts. The exit is always on foot.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/transit/15.16,48.207?contours_minutes=45&access=car" | jq '.metadata | {access, access_stops_improved, sites: .access_sites[:3]}'
```

| Parameter | Values | Meaning |
|---|---|---|
| `access` | `foot` (default), `bike`, `car` | Mode to the first stop |
| `park` | `any` (default), `official` | `official`: bike only at stops within 200 m of a B+R site, car only at official P+R sites |
| `sites` | 1 to 50 (default 10) | Entries in `metadata.access_sites` |

`metadata.access_sites` lists the accesses that beat walking, sorted by travel time: `source` (`site` for an official P+R, `stop` for parking at the stop), `kind`, `name`, `lon`, `lat`, `arrive_min` (travel time), `ready_min` (ready to board, incl. parking and walk) and `stops_won`. Bringing the bike on the train (`egress=bike`) is not offered: the timetable data carry no bike-carriage information.

## Show departures in an app

1. Search a stop with [`/v1/stops`](https://docs.mapx.at/api/transit.md) or let the user click the map.
2. Call `/v1/departures` and refresh every minute.
3. Show line, destination and `time`; label the times as "scheduled" (timetable data).

## Next

- [Transit API](https://docs.mapx.at/api/transit.md)
- [Navigation](https://docs.mapx.at/guides/navigation.md)
- [Attribution](https://docs.mapx.at/reference/attribution.md): timetable data credit

# Traffic guide: incidents and layer

Source: https://docs.mapx.at/guides/traffic/

> Show traffic incidents, motorway speed limits and a traffic layer on the map and route around closures, with the mapx traffic API.

Traffic data covers the motorway and expressway network (ASFINAG, EVIS.AT) and planned roadworks on other roads. Requests are in the `search` zone, 10 per second, and need a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); the samples send it in the `Authorization` header, which browsers may send too.

| Task | Endpoint |
|---|---|
| List incidents, roadworks and closures | [`/v1/traffic/incidents`](https://docs.mapx.at/api/traffic.md) |
| Speed limits shown by variable message signs on motorways (no congestion data) | [`/v1/traffic/sign-limits`](https://docs.mapx.at/api/traffic.md) |
| Positions of the signs | [`/v1/traffic/signs`](https://docs.mapx.at/api/traffic.md) |
| Draw traffic on a map | `/v1/traffic/tiles.json` |
| Check whether the data is fresh | `/v1/traffic/status` |
| Route around closures | profile `driving-traffic` (`driving` also avoids binding closures) |

## List incidents

```js
const url = "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&type=road_closure,construction&limit=20";
const { features, total, next } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
for (const f of features) console.log(f.properties.road, f.properties.type, f.properties.text);
```

Filters: `bbox`, `road` (`A1`, `A 1` and `A01` are the same road; ramps and branches such as `road_section` `A13.33.4` belong to `A13`), `road_class` (`motorway`, `expressway`, `federal`, `state`, comma-separated, for A, S, B and L roads), `type` (`accident`, `congestion`, `construction`, `lane_restriction`, `road_closure`, `road_hazard`, `miscellaneous`), `planned=true|false`, and a time window with `from` and `to` (a date, or a date-time with `Z` or an offset such as `2026-12-01T10:00:00+01:00`; `to` before `from` returns `400`). By default only active reports are returned; `active=false` adds ended and future ones. A report with `mandatory: true` is binding; show its text unchanged.

Results are sorted by importance: closures and binding reports first, then motorways, expressways, federal and state roads, then by start. Page with `limit` (default 500, maximum 1000): `next` is the full URL of the next page (with `cursor`); follow it until `next` is `null`. With `POST`, `next` carries only the `cursor`: send the same body to that URL. `total` is the number of matches. Bad values for `road`, `road_class`, `type` or `cursor` return a `400` problem response.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/incidents?road=A1&road_class=motorway&limit=100" | jq '.total, .next'
```

Every report has `type` and `impact` (`critical` closed for cars, `major` binding, lane restriction or accident, `minor` other unplanned, `low` other planned), `kind` (detailed category as an English enum after DATEX II: `road_closure`, `winter_closure`, `tunnel_closure`, `ramp_closure`, `lane_closure`, `roadworks`, `maintenance_works`, `repair_works`, `resurfacing_works`, `safety_upgrade_works`, `grass_cutting`, `alternating_traffic`, `temporary_traffic_lights`, `traffic_management`, `speed_restriction`, `congestion`, `accident`, `obstruction`, `hazardous_driving_conditions`, `weather`, `safety_exercise`, `route_recommendation`, `event`, `other`; the same values as `category` of the closures in directions, matrix, isochrone and optimized-trips), `road` (road number, `null` for municipal streets) and from EVIS `road_name`, `direction` (direction as text), `regions` (federal states), `creator` and `transport_modes`. `closed: true` means closed for cars; a `road_closure` with `closed: false` applies to other modes only (e.g. pedestrians, cycles) or traffic is led over the opposite carriageway.

## Lane details on motorways

Roadworks on motorways and expressways usually carry `lanes` from ASFINAG: one entry per direction and carriageway with the closed lanes (`right`, `middle`, `left`, `hard_shoulder`), the number of open lanes (`open`) out of `original`, `contraflow`, `road_closed` and the temporary `speed_limit_kmh`. `direction` is `aligned` (with increasing road kilometers) or `opposite`. `sources` lists where a report comes from (`evis`, `asfinag`); `source` names the dataset (`evis-planned+asfinag` when both match, `asfinag-roadworks` for roadworks only ASFINAG reports). The text of an EVIS report is never changed.

```js
const url = "https://api.mapx.at/v1/traffic/incidents?road=A12&type=construction";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
for (const f of features) for (const l of f.properties.lanes ?? []) console.log(f.properties.road, l.closed, l.open, l.speed_limit_kmh);
```

## Incidents along a route

Pass the route as `route` (encoded polyline, precision via `route_precision` 5 or 6, or a GeoJSON LineString). Results are sorted by position on the route. For long routes use `POST` with a JSON body.

```bash
# route Vienna–Graz as polyline6, then the reports in a 50 m corridor
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.4395,47.0707?geometries=polyline6" \
  | jq '{route: .routes[0].geometry, route_precision: "6", type: "road_closure,construction", buffer: 50}' > body.json
curl -s -X POST -H "Authorization: Bearer YOUR_TOKEN" -H "Content-Type: application/json" https://api.mapx.at/v1/traffic/incidents --data @body.json | jq '.total'
```

`buffer` is the corridor half-width in meters (1 to 1000, default 25).

## Show traffic on the map

```js
map.on("load", () => {
  map.addSource("mx-traffic", { type: "vector", url: "https://api.mapx.at/v1/traffic/tiles.json" });
  // layer "sign_limits": speed limits shown by variable message signs, per motorway section and direction
  map.addLayer({ id: "mx-sign-limits", type: "line", source: "mx-traffic", "source-layer": "sign_limits",
                 paint: { "line-width": 4, "line-color": "#3a5fcd" } });
  // layer "incidents": reports and closures (lines and points)
  map.addLayer({ id: "mx-incidents-lines", type: "line", source: "mx-traffic", "source-layer": "incidents",
                 filter: ["!=", ["geometry-type"], "Point"],
                 paint: { "line-width": 5, "line-color": "#b91c1c", "line-dasharray": [2, 1] } });
  map.addLayer({ id: "mx-incidents-points", type: "circle", source: "mx-traffic", "source-layer": "incidents",
                 filter: ["==", ["geometry-type"], "Point"],
                 paint: { "circle-radius": 6, "circle-color": "#b91c1c", "circle-stroke-color": "#fff", "circle-stroke-width": 1.5 } });
});
```

The vector source has two layers: `sign_limits` (speed limits shown by variable message signs, with `speed_limit`, `normal_speed` and `reduction` in km/h; this is not a traffic state) and `incidents` (reports and closures with `id`, `type`, `impact`, `start`, `end`). The token is passed with `transformRequest` like on every map, see [Access and limits](https://docs.mapx.at/start/access-and-limits.md). The mapx styles (for example `streets`) already use the source name `traffic`, so the source is called `mx-traffic` here. The styles `traffic-day` and `traffic-night` (night map with colors brightened for a dark background) draw both layers ready-made, the speed limits in a neutral color. Tiles are in the `static` zone (5 per second); let the client cache them.

## Route around closures

```
GET /v1/directions/driving-traffic/{coordinates}
```

The route avoids the current binding closures from the ASFINAG and EVIS.AT reports, discourages conditional closures and lists the reports along the way in `legs[].incidents`. Network-wide traffic flow (measured speeds, congestion on all roads) is not taken into account. Use `incidents=true` to get the list with other car profiles.

## Next

- [Traffic API](https://docs.mapx.at/api/traffic.md)
- [Navigation](https://docs.mapx.at/guides/navigation.md): tolls, trucks, alternatives
- [Attribution](https://docs.mapx.at/reference/attribution.md): credit ASFINAG and GIP.at

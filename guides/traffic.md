# Traffic

Incident reports, motorway speed limits, a traffic layer on the map and traffic-aware routes.

Traffic data covers the motorway and expressway network (ASFINAG, EVIS.AT) and planned roadworks on other roads. Requests are in the `search` zone, 10 per second, and need a token ([Access and limits](/start/access-and-limits/)); `curl` samples send it as a header, browser samples as `access_token`. The map source below takes the token from the `transformRequest` of [First map](/start/first-map/).

| Task | Endpoint |
|---|---|
| List incidents, roadworks and closures | [`/v1/traffic/incidents`](/api/traffic/) |
| Speed limits on variable message signs | [`/v1/traffic/flow`](/api/traffic/) |
| Positions of the signs | [`/v1/traffic/signs`](/api/traffic/) |
| Draw traffic on a map | `/v1/traffic/tiles.json` |
| Check whether the data is fresh | `/v1/traffic/status` |
| Route around closures | profile `driving-traffic` |

## List incidents

```js
const url = "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&type=road_closure,construction&limit=20&access_token=YOUR_TOKEN";
const { features, total, next_cursor } = await (await fetch(url)).json();
for (const f of features) console.log(f.properties.road, f.properties.type, f.properties.text);
```

Filters: `bbox`, `road` (`A1`, `A 1` and `A01` are the same road), `road_class` (`motorway`, `expressway`, `federal`, `state`, comma-separated, for A, S, B and L roads), `type` (`accident`, `congestion`, `construction`, `lane_restriction`, `road_closure`, `road_hazard`, `miscellaneous`), `planned=true|false`, and a time window with `from` and `to` (UTC). By default only active reports are returned; `active=false` adds ended and future ones. A report with `mandatory: true` is binding; show its text unchanged.

Results are sorted by importance: closures and binding reports first, then motorways, expressways, federal and state roads, then by start. Page with `limit` (default 500, maximum 1000) and `cursor`: pass `next_cursor` from the response, or follow `next`, until `next` is `null`. `total` is the number of matches. Bad values for `road`, `road_class`, `type` or `cursor` return a `400` problem response.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/incidents?road=A1&road_class=motorway&limit=100" | jq '.total, .next'
```

## Lane details on motorways

Roadworks on motorways and expressways carry `lanes` from ASFINAG: one entry per direction and carriageway with the closed lanes (`right`, `middle`, `left`, `hard_shoulder`), the number of open lanes (`open`) out of `original`, `contraflow`, `road_closed` and the temporary `speed_limit_kmh`. `direction` is `aligned` (with increasing road kilometers) or `opposite`. `sources` lists where a report comes from (`evis`, `asfinag`); `source` names the dataset (`evis-planned+asfinag` when both match, `asfinag-roadworks` for roadworks only ASFINAG reports). The text of an EVIS report is never changed.

```js
const url = "https://api.mapx.at/v1/traffic/incidents?road=A12&type=lane_restriction&access_token=YOUR_TOKEN";
const { features } = await (await fetch(url)).json();
for (const f of features) for (const l of f.properties.lanes) console.log(f.properties.road, l.closed, l.open, l.speed_limit_kmh);
```

## Incidents along a route

Pass the route as `route` (encoded polyline or GeoJSON). Results are sorted by position on the route. For long routes use `POST` with a JSON body.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" -X POST https://api.mapx.at/v1/traffic/incidents -H "Content-Type: application/json" -d '{
  "route": { "type": "LineString", "coordinates": [[16.37, 48.21], [16.3, 48.19]] },
  "type": "road_closure,construction", "buffer": 50
}' | jq '.total'
```

`buffer` is the corridor half-width in meters (1 to 1000, default 25).

## Show traffic on the map

```js
map.on("load", () => {
  map.addSource("traffic", { type: "vector", url: "https://api.mapx.at/v1/traffic/tiles.json" });
  map.addLayer({ id: "traffic", type: "line", source: "traffic", "source-layer": "traffic",
                 paint: { "line-width": 4, "line-color": "#e11d48" } });
});
```

The style `traffic-day` already contains the layer. Tiles are in the `static` zone (5 per second); let the client cache them.

## Route with traffic

```
GET /v1/directions/driving-traffic/{coordinates}
```

The route avoids current closures and lists incidents in `legs[].incidents`. Use `incidents=true` to get the list with other car profiles.

## Next

- [Traffic API](/api/traffic/)
- [Navigation](/guides/navigation/): tolls, trucks, alternatives
- [Attribution](/reference/attribution/): credit ASFINAG and GIP.at
# Traffic

Incident reports, motorway speed limits, a traffic layer on the map and traffic-aware routes.

Traffic data covers the motorway and expressway network (ASFINAG, EVIS.AT) and planned roadworks on other roads. Requests are in the `search` zone, 10 per second.

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
const url = "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&type=road_closure,construction&limit=20";
const { features, total, next_offset } = await (await fetch(url)).json();
for (const f of features) console.log(f.properties.road, f.properties.type, f.properties.text);
```

Filters: `bbox`, `road` (for example `A12`), `type` (`accident`, `congestion`, `construction`, `lane_restriction`, `road_closure`, `road_hazard`, `miscellaneous`), `planned=true|false`, and a time window with `from` and `to` (UTC). By default only active reports are returned; `active=false` adds ended and future ones. Page with `limit` and `offset`. A report with `mandatory: true` is binding; show its text unchanged.

## Incidents along a route

Pass the route as `route` (encoded polyline or GeoJSON). Results are sorted by position on the route. For long routes use `POST` with a JSON body.

```bash
curl -s -X POST https://api.mapx.at/v1/traffic/incidents -H "Content-Type: application/json" -d '{
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
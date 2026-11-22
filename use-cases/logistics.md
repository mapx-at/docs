# Logistics: truck routing and matrix

Source: https://docs.mapx.at/use-cases/logistics/

> Truck routing with vehicle size, travel-time matrices, tour planning and traffic for deliveries and fleets, built on the mapx APIs for Austria.

All requests need a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); in server code use a secret token (`mx_sk_…`) and never ship it in a web page.

Fleet and delivery software needs a route that a truck can really drive, travel times for many pairs, a good order of stops and the current traffic reports. All four are one request each. Coordinates are `lon,lat`, times are seconds, distances metres.

## Truck routes

The profile `driving-truck` takes the vehicle size and avoids roads with matching weight, height, width and length limits and bans. Without parameters it assumes 40 t, 4 m high and 2.55 m wide.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/driving-truck/15.6256,48.2049;15.3294,48.2272?weight=40&height=4&width=2.55&length=16.5&truck_type=semi&overview=false" \
  | jq '.routes[0] | {distance, duration, truck, truck_go_distance: .toll.truck_go_distance}'
```

`routes[0].toll.truck_go_distance` is the length subject to the GO toll; per step it is `truck_go_distance`. Speeds follow truck limits (80 km/h on motorways). District-wide bans from free-text ordinances are not routed: affected roads come as `steps[].notices[]` with `type: "truck_restriction"`; show them to the dispatcher. For cars, `exclude=toll` and `exclude=motorway` avoid toll sections and motorways (profiles `driving`, `driving-traffic` and `driving-truck`); `exclude=ferry` works for every profile except `driving-traffic`.

In JavaScript:

```js
const q = new URLSearchParams({ weight: 40, height: 4, width: 2.55, length: 16.5, truck_type: "semi", steps: true });
const r = await (await fetch(`https://api.mapx.at/v1/directions/driving-truck/15.6256,48.2049;15.3294,48.2272?${q}`, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
const notices = r.routes[0].legs.flatMap((l) => l.steps).flatMap((s) => s.notices ?? []);
```

## Travel times for many pairs

`matrix` answers all pairs in one request, up to the limits of the [Matrix API](https://docs.mapx.at/api/matrix.md). Use `sources` and `destinations` (indices) to ask for one row only.

```js
const depots = [[16.3725, 48.2085], [15.3294, 48.2272], [14.2858, 48.3069]];
const q = depots.map((p) => p.join(",")).join(";");
const m = await (await fetch(`https://api.mapx.at/v1/matrix/driving/${q}?annotations=duration,distance`, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
console.log(m.durations[0][1] / 60, "min", m.distances[0][1] / 1000, "km"); // Vienna to Melk
```

## Plan the tour

`POST /v1/optimization` assigns jobs to vehicles and orders them: capacity, time windows, service times (seconds), start and end per vehicle. At most 10 vehicles, 50 jobs and 60 distinct locations per request. Secret token (`mx_sk_…`) from a server only; a public token gets `403 pk-not-allowed`.

```js
const body = {
  profile: "driving",
  vehicles: [{ id: 1, start: [16.3725, 48.2083], end: [16.3725, 48.2083], time_window: [28800, 61200] }],
  services: [
    { id: 1, location: [15.6256, 48.2049], service: 300, time_windows: [[32400, 43200]] },
    { id: 2, location: [14.2858, 48.3069], service: 600 },
  ],
};
const res = await fetch("https://api.mapx.at/v1/optimization", { method: "POST", headers: { "content-type": "application/json", Authorization: "Bearer YOUR_TOKEN" }, body: JSON.stringify(body) });
const tour = await res.json();
console.log(tour.routes[0].steps.map((s) => s.id ?? s.type), tour.unassigned); // unassigned: jobs that did not fit
```

Draw the result with one `directions` request through `tour.routes[0].steps` locations (up to 25 points); see the [tour planning example](https://docs.mapx.at/examples/optimization.md).

## Traffic

`driving-traffic` uses the reports of ASFINAG and EVIS.AT: binding closures are avoided, non-binding closures are avoided where possible, and on motorways the speed limits currently shown on variable message signs slow down the travel time. There is no area-wide measured traffic flow (speeds on every road, congestion as "flow"); reported jams without a closure do not change the route, they only show up as incidents. Incidents on the route come with `incidents=true` in `legs[].incidents`. For a region, list the active incidents or show the traffic style:

```js
const bbox = [14.5, 47.8, 16.9, 48.6]; // Lower Austria
const { features } = await (await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${bbox}&active=true&type=construction&limit=50`, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
console.log(features.length, features[0]?.properties.road, features[0]?.properties.text);
```

The map style `traffic-day` shows the current reports and closures and, on motorways, the sections with a speed limit from variable message signs (no area-wide traffic flow); see the [example](https://docs.mapx.at/examples/traffic.md). Traffic data: ASFINAG / EVIS.AT (CC BY 4.0).

## Next steps

- [Navigation](https://docs.mapx.at/guides/navigation.md): all profiles and parameters
- [Traffic](https://docs.mapx.at/guides/traffic.md)
- [Errors and limits](https://docs.mapx.at/api/errors.md): rate-limit zones and what to do on `429`

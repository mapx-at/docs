# Navigation

Routes for car, truck, bike and foot, tolls, isochrones, matrices, map matching and tour planning.

Routing runs on the Austrian road graph (GIP.at). Coordinates are `lon,lat`, joined with `;`.

| Task | Endpoint | Zone |
|---|---|---|
| Route between 2 to 25 points | `/v1/directions/{profile}/{coordinates}` | `route` |
| Area reachable in N minutes | `/v1/isochrone/{profile}/{coordinate}` | `heavy` |
| Travel times between many points | `/v1/matrix/{profile}/{coordinates}` | `heavy` |
| Snap a GPS trace to roads | `/v1/matching/{profile}/{coordinates}` | `heavy` |
| Best order of stops | `/v1/optimized-trips/{profile}/{coordinates}` | `heavy` |
| Tours for several vehicles | `POST /v1/optimization` | `heavy` |

## Profiles

| Profile | For |
|---|---|
| `driving` | Car |
| `driving-traffic` | Car with current traffic and closures |
| `driving-truck` | Trucks over 3.5 t |
| `cycling` | Bike: prefers cycle paths, lanes and signposted routes, rates unpaved sections and steep grades worse |
| `cycling-mtb` | Mountain bike: prefers forest tracks and singletrails, considers the MTB difficulty |
| `walking` | On foot |

Public transport is a separate API: [Transit](/guides/transit/).

## A route

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=true&language=en";
const { routes } = await (await fetch(url)).json();
const r = routes[0];
console.log(r.distance, "m", r.duration, "s");
```

`overview=simplified` (default) returns a light geometry, `full` every vertex, `false` none. `steps=true` adds turn-by-turn steps. `alternatives=true` adds up to two more `routes[]`. `annotations=duration,distance,speed,maxspeed,elevation` adds values per segment. With `cycling_surface=any` (default) unpaved sections are rated worse but stay allowed. Heights inside bridges and tunnels are not taken from the terrain model. Cycling and walking steps carry `grade_pct`; the route summary has ascent, descent and elevation range.

## Truck routes

```bash
curl -s "https://api.mapx.at/v1/directions/driving-truck/15.6256,48.2049;15.3294,48.2272?weight=40&height=4&width=2.55&length=16.5&truck_type=semi&overview=false" | jq '.routes[0] | {distance, duration, truck}'
```

The router avoids roads closed to the vehicle: weight-class and trailer bans, height, width, weight and length limits. Speeds are capped at 80 km/h on motorways, 70 on rural roads, 50 in built-up areas. District-wide bans from free-text ordinances are not routed; affected roads are reported in `steps[].notices[]` with `type: truck_restriction`.

## Tolls

Car routes include `toll_road`, `toll_road_m` and `toll` with vignette products and special toll sections. Truck routes report the kilometers subject to the GO toll in `routes[].toll.truck_go_km`. Avoid tolls or motorways:

```
.../directions/driving/{coordinates}?exclude=toll
.../directions/driving/{coordinates}?exclude=motorway,toll
```

Vignette roads remain allowed with `exclude=toll`. Toll source: ASFINAG, CC BY 4.0.

`exclude` works for `driving`, `driving-traffic` and `driving-truck`. `ferry` is not supported.

## Closures

Binding full closures from the traffic reports (EVIS/ASFINAG) are avoided by `driving` and `driving-traffic`. The roads concerned are blocked for the route search; `routes[].closures_avoided` lists the closures that the route without closures would have driven through (`id`, `text`, `road`, `category`). If a closure cannot be avoided, the route runs through it and `routes[].closures_unavoidable` lists it. Conditional closures (weekdays, hours in the text) only discourage a road and are not listed. Source: ASFINAG, CC BY 4.0; show the texts unchanged.

```bash
curl -s "https://api.mapx.at/v1/directions/driving/14.2,48.27;14.45,48.30?overview=false" | jq '.routes[0].closures_avoided'
```

## Voice and banner instructions

With `steps=true` you can add `voice_instructions=true` and `banner_instructions=true` (driving profiles). The format follows Mapbox: each step carries the instructions for the maneuver at its end.

```bash
curl -s "https://api.mapx.at/v1/directions/driving/15.6256,48.2049;15.3294,48.2272?steps=true&voice_instructions=true&banner_instructions=true&language=en&overview=false" | jq '.routes[0].legs[0].steps[1] | {voiceInstructions, bannerInstructions}'
```

- `voiceInstructions[]`: `distanceAlongGeometry` (meters before the end of the step at which to speak), `announcement` and `ssmlAnnouncement`. Announcement distances depend on speed: on motorways 2 km, 1 km and 300 m ahead, on rural roads 1 km, 400 m and 120 m, in town 400 m and 150 m; the last one is spoken without a distance. The first step also announces the departure. `language=de|en` selects the language.
- `bannerInstructions[]`: `primary` and optional `secondary` with `text`, `components[]` (`type` `icon` for the road number, `text` for names and destination signage), `type`, `modifier`. With destination signage, the destination is the primary text and the road the secondary line.

## Isochrones, matrix, matching

```js
// Where can I walk in 5, 10 and 15 minutes?
const iso = await (await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15")).json();

// Travel times from the first point to the others
const m = await (await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?sources=0&annotations=duration,distance")).json();
```

The isochrone result is a `FeatureCollection` of polygons, largest first. Show it with a `fill` layer; see [Isochrone](/examples/isochrone/) and [Matrix](/examples/matrix/). Map matching takes a GPS trace and returns the matched route.

## Tour planning

`optimized-trips` orders up to 25 stops (round trip or fixed start and end). `POST /v1/optimization` plans tours for several vehicles with capacities and time windows.

```bash
curl -s -X POST https://api.mapx.at/v1/optimization -H "Content-Type: application/json" -d '{
  "vehicles": [{ "id": 1, "start": [16.3725, 48.2083], "end": [16.3725, 48.2083] }],
  "services": [{ "id": 1, "location": [15.6256, 48.2049], "service": 300 },
               { "id": 2, "location": [14.2858, 48.3069], "service": 300 }]
}' | jq '.summary'
```

## Handle failures

`422` means the request is valid but unanswerable, for example no route between the points. `400` carries the faulty parameter in `param`. Retry `429` after `Retry-After` seconds. See [Errors and limits](/api/errors/).

## Next

- [Directions API](/api/directions/), [Optimization API](/api/optimization/)
- [Traffic](/guides/traffic/): incidents along a route
- [Route example](/examples/route/)
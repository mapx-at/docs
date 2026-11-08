# Tourism

The snippets that use `mapx.…` expect `mapx.setAccessToken("YOUR_TOKEN")` once ([Access and limits](/start/access-and-limits/)).

Visitors ask "Which way by bike, how steep is it, and what is the weather?". The answers come from the same API: routes with the profiles `cycling` and `cycling-mtb`, a profile of the heights, the signposted cycle routes on the map and the forecast for the destination.

## Routes for bikes

`cycling` prefers cycle paths and signposted routes and rates unpaved sections and steep grades worse. `cycling-mtb` prefers forest tracks and singletrails. With `steps=true` every step carries the type of infrastructure and the surface, `routes[].summary` has ascent, descent and maximum grade.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/cycling-mtb/15.3294,48.2272;15.4,48.3?steps=true&overview=false" \
  | jq '.routes[0] | {distance, duration, summary}'
```

```js
const r = await mapx.directions("cycling", [[15.3294, 48.2272], [15.6141, 48.4097]], { steps: true, alternatives: true, annotations: ["elevation"] });
console.log(r.routes.map((x) => `${(x.distance / 1000).toFixed(1)} km, up ${x.summary?.ascent_m} m`));
```

## Show the cycle routes

The `streets` style contains the signposted cycle and MTB routes as the layer group `bike_routes`. Switch it on with the style metadata, or let the user toggle it:

```js
const map = mapx.map({ accessToken: "YOUR_TOKEN", container: "map", style: "streets", center: [15.45, 48.3], zoom: 10 });
map.addControl(new mapx.LayerControl({ groups: ["bike_routes", "relief"] }), "top-right");
map.addControl(new mapx.RouteControl({ profiles: ["cycling", "cycling-mtb", "walking"] }), "top-right");
```

The [SDK example](/examples/sdk-controls/) shows both controls together.

## Elevation profile

Every tour page benefits from a profile. Feed the route geometry (2 to 512 points; the default simplified route geometry fits) into the elevation API:

```js
const route = (await (await fetch("https://api.mapx.at/v1/directions/cycling/15.3294,48.2272;15.6141,48.4097?access_token=YOUR_TOKEN")).json()).routes[0];
// the default overview=simplified keeps the line below the 512-point limit
const p = await mapx.elevationProfile(route.geometry.coordinates, 120);
console.log(`up ${Math.round(p.ascent)} m, down ${Math.round(p.descent)} m, ${Math.round(p.min)}-${Math.round(p.max)} m`);
// p.distances[i] (m from start) and p.elevations[i] (m) give the chart
```

The height model is the official DTM of the BEV. See [Elevation profile](/guides/elevation-profile/) for a finished chart and the [live example](/examples/elevation-profile/).

## Reachability for the area

Which destinations can guests reach in 30 minutes by bike? One isochrone request:

```js
const iso = await mapx.isochrone("cycling", [15.3294, 48.2272], { minutes: [15, 30, 45] }); // largest area first
map.addSource("reach", { type: "geojson", data: iso });
map.addLayer({ id: "reach", type: "fill", source: "reach", paint: { "fill-color": "#2346d8", "fill-opacity": 0.12 } });
```

For public transport to the trailhead use [`transitDirections`](/guides/sdk/helpers/) with `bike: true` to ride to the stop.

## Weather for the destination

```js
const f = await mapx.weatherForecast([12.694, 47.074], { hours: 12 }); // Grossglockner
console.log(f.hours.map((h) => `${h.time.slice(11, 16)} ${h.temperature_c}°C ${h.precipitation_mm} mm`).join("\n"));
```

`weatherCurrent` gives the measurements of the nearest stations. As a map layer see the [weather overlay](/examples/weather-overlay/). Weather data: GeoSphere Austria (CC BY 4.0).

## Next steps

- [Navigation](/guides/navigation/) and [Elevation and terrain](/guides/elevation/)
- [3D terrain](/guides/3d-terrain/) for a spectacular tour map
- [Static images](/guides/static-images/) for brochures and newsletters


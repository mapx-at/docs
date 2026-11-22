# Tourism: bike routes and elevation profile

Source: https://docs.mapx.at/use-cases/tourism/

> Bike and mountain-bike routes, cycle-route layer, elevation profile and weather for tours and visitor information, built on the mapx APIs.

All requests need a token; replace `YOUR_TOKEN` with your own ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)).

Visitors ask "Which way by bike, how steep is it, and what is the weather?". The answers come from the same API: routes with the profiles `cycling` and `cycling-mtb`, a profile of the heights, the signposted cycle routes on the map and the forecast for the destination.

## Routes for bikes

`cycling` prefers cycle paths and signposted routes and rates unpaved sections and steep grades worse. `cycling-mtb` prefers forest tracks and singletrails. With `steps=true` every step carries the type of infrastructure and the surface, `routes[].summary` has ascent, descent and maximum grade.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/cycling-mtb/15.3294,48.2272;15.4,48.3?steps=true&overview=false" \
  | jq '.routes[0] | {distance, duration, summary}'
```

```js
const r = await (await fetch("https://api.mapx.at/v1/directions/cycling/15.3294,48.2272;15.6141,48.4097?steps=true&alternatives=true&annotations=elevation", { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
console.log(r.routes.map((x) => `${(x.distance / 1000).toFixed(1)} km, up ${x.elevation_summary?.ascent} m`));
```

## Show the cycle routes

The `streets` style contains the signposted cycle and MTB routes as the layer group `bike_routes`. Switch it on with the style metadata, or let the user toggle it:

```js
const TOKEN = "YOUR_TOKEN";
// the style uses PMTiles sources: load pmtiles.js as in the examples
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map", style: "https://api.mapx.at/v1/styles/streets", center: [15.45, 48.3], zoom: 10,
  transformRequest: (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url }),
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
map.on("load", () => {
  // switch on the layers of the group bike_routes (off by default in the style)
  for (const id of map.getStyle().metadata["mapx:groups"].bike_routes) map.setLayoutProperty(id, "visibility", "visible");
});
```

The example [Route](https://docs.mapx.at/examples/route.md) calculates and draws a route on the map (MapLibre, OpenLayers, Leaflet).

## Elevation profile

Every tour page benefits from a profile. Feed the route geometry (2 to 512 points; the default simplified route geometry fits) into the elevation API:

```js
const route = (await (await fetch("https://api.mapx.at/v1/directions/cycling/15.3294,48.2272;15.6141,48.4097", { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json()).routes[0];
// the default overview=simplified keeps the line below the 512-point limit
const line = route.geometry.coordinates.map((c) => c.join(",")).join(";");
const p = (await (await fetch(`https://api.mapx.at/v1/elevation/profile?coordinates=${line}&samples=120`, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json()).properties;
console.log(`up ${Math.round(p.ascent)} m, down ${Math.round(p.descent)} m, ${Math.round(p.min_elevation)}-${Math.round(p.max_elevation)} m`);
// p.distances[i] (m from start) and p.elevations[i] (m) give the chart
```

The height model is the official DTM of the BEV. See [Elevation profile](https://docs.mapx.at/guides/elevation-profile.md) for a finished chart and the [live example](https://docs.mapx.at/examples/elevation-profile.md).

## Reachability for the area

Which destinations can guests reach in 30 minutes by bike? One isochrone request:

```js
const iso = await (await fetch("https://api.mapx.at/v1/isochrone/cycling/15.3294,48.2272?contours_minutes=15,30,45", { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json(); // largest area first
map.addSource("reach", { type: "geojson", data: iso });
map.addLayer({ id: "reach", type: "fill", source: "reach", paint: { "fill-color": "#2346d8", "fill-opacity": 0.12 } });
```

For public transport to the trailhead use the profile `bike_transit` (`GET /v1/directions/bike_transit/{start};{end}`, [Transit](https://docs.mapx.at/guides/transit.md)) to ride to the stop.

## Weather for the destination

```js
const f = await (await fetch("https://api.mapx.at/v1/weather/forecast?lon=12.694&lat=47.074&hours=12", { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json(); // Grossglockner
console.log(f.hours.map((h) => `${new Date(h.time).toLocaleTimeString("en-GB", { hour: "2-digit", minute: "2-digit", timeZone: "Europe/Vienna" })} ${h.temperature_c}°C ${h.precipitation_mm} mm`).join("\n"));
```

`GET /v1/weather/current?lon=…&lat=…` gives the measurements of the nearest stations. As a map layer see the [weather overlay](https://docs.mapx.at/examples/weather-overlay.md). Weather data: GeoSphere Austria (CC BY 4.0).

## Next steps

- [Navigation](https://docs.mapx.at/guides/navigation.md) and [Elevation and terrain](https://docs.mapx.at/guides/elevation.md)
- [3D terrain](https://docs.mapx.at/guides/3d-terrain.md) for a spectacular tour map
- [Static images](https://docs.mapx.at/guides/static-images.md) for brochures and newsletters

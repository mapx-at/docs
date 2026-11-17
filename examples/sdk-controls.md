# SDK example: route planner and layers

Source: https://docs.mapx.at/examples/sdk-controls/

> RouteControl with truck profile, LayerControl, search and style switcher: ready-made map controls of the mapx-gl SDK in one runnable example.

Five controls of the [SDK `mapx-gl`](https://docs.mapx.at/guides/sdk/index.md) give a complete map tool. `RouteControl` (the arrow button) offers the six profiles, avoid tolls and motorways and truck size; `LayerControl` builds its switches from the style metadata.

## Code

````js
// mapx-gl SDK: route planner (car, truck, bike, tolls, truck size), layer switcher from the style, search in a few lines.
import mapx from "https://map.mapx.at/sdk/v1/mapx-gl.bundle.js";

const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at

const map = mapx.map({ accessToken: TOKEN, container: "map", style: "streets", center: [15.33, 48.23], zoom: 11 });
map.addControl(new mapx.SearchControl(), "top-left");
map.addControl(new mapx.RouteControl({ profile: "driving-truck" }), "top-right");
map.addControl(new mapx.LayerControl({ groups: ["buildings", "parcels", "relief", "bike_routes"] }), "top-right");
map.addControl(new mapx.StyleSwitcherControl(), "top-right");
````

## APIs used

- [Directions](https://docs.mapx.at/api/directions.md) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
- [Search](https://docs.mapx.at/api/geocoding.md) `GET api.mapx.at/v1/geocode`

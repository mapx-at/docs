# SDK: Routenplaner und Ebenen

RouteControl mit Lkw-Profil, LayerControl, Suche und Stilumschalter.

Fünf Controls des [SDK `mapx-gl`](/de/guides/sdk/) ergeben ein vollständiges Kartenwerkzeug. `RouteControl` (Pfeil-Schaltfläche) bietet die sechs Profile, Maut und Autobahn meiden und Lkw-Maße; `LayerControl` baut seine Schalter aus den Style-Metadaten.

```js
// mapx-gl SDK: route planner (car, truck, bike, tolls, truck size), layer switcher from the style, search in a few lines.
import mapx from "mapx-gl-sdk";

const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at

const map = mapx.map({ accessToken: TOKEN, container: "map", style: "streets", center: [15.33, 48.23], zoom: 11 });
map.addControl(new mapx.SearchControl(), "top-left");
map.addControl(new mapx.RouteControl({ profile: "driving-truck" }), "top-right");
map.addControl(new mapx.LayerControl({ groups: ["buildings", "parcels", "relief", "bike_routes"] }), "top-right");
map.addControl(new mapx.StyleSwitcherControl(), "top-right");
```

## Verwendete APIs

- [Directions](/de/api/directions/) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
- [Geocoding](/de/api/geocoding/) `GET api.mapx.at/v1/geocode`
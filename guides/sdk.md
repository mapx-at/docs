# SDK mapx-gl

A small add-on for MapLibre GL JS: map factory, search, style switcher, measuring, elevation profile, 3D terrain and API helpers.

`mapx-gl` is a small add-on (ESM, TypeScript) for MapLibre GL JS. It sets the style URLs, registers the `pmtiles://` protocol and adds ready-made controls and helpers for the [APIs](/api/). Peer dependencies: `maplibre-gl@{V.maplibre}` and `pmtiles@{V.pmtiles}`.

## Install

Download the files from `https://map.mapx.at/sdk/v1/` and serve them with your page:

| File | Use |
|---|---|
| `mapx-gl.bundle.js` | Everything included, load as `<script type="module">` |
| `mapx-gl.js` | Without peers, for projects with their own build |
| `mapx-gl.css` | Styles for the controls, load after `maplibre-gl.css` |

```html
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
<link rel="stylesheet" href="./mapx-gl.css">
<div id="map" style="height:100vh"></div>
<script type="module">
  import mapx from "./mapx-gl.bundle.js";
  const map = mapx.map({ container: "map", style: "streets", center: [15.33, 48.23], zoom: 12 });
  map.addControl(new mapx.SearchControl(), "top-left");
  map.addControl(new mapx.StyleSwitcherControl(), "top-right");
</script>
```

A complete example is at [map.mapx.at/sdk/v1/example.html](https://map.mapx.at/sdk/v1/example.html).

## Map

`mapx.map(options)` returns a MapLibre map. `style` is a short name (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`) that becomes `https://api.mapx.at/v1/styles/{id}`, or any style URL or object. `terrain: true` turns on 3D terrain.

## Controls

| Control | Function | API |
|---|---|---|
| `SearchControl` | Search box with result list | [Geocoding](/api/geocoding/) |
| `StyleSwitcherControl` | Switch between styles | [Styles](/api/styles/) |
| `MeasureControl` | Measure distances | none |
| `ElevationProfileControl` | Elevation profile of a drawn line | [Elevation](/api/elevation/) |
| `Terrain3DControl` | 3D terrain on and off | [Tiles](/api/tiles/) |

## Helpers

| Helper | Returns |
|---|---|
| `mapx.geocode(q, { limit, types, bbox, proximity })` | List of features |
| `mapx.directions(profile, [[lng, lat], ...], { steps, overview })` | Directions response |
| `mapx.isochrone(profile, [lng, lat], { minutes })` | GeoJSON polygons |
| `mapx.elevation(point or points, model)` | Height or list of heights in m |
| `mapx.elevationProfile(line, samples, model)` | Distances, heights, ascent, descent |
| `mapx.setApiBase(url)` | Use another API host, for example for tests |

```js
const r = await mapx.directions("walking", [[15.33, 48.22], [15.34, 48.23]]);
const h = await mapx.elevation([12.694, 47.074]); // 3726.3
```

## Next

- [Examples](/examples/)
- [Maps](/guides/maps/) and [Navigation](/guides/navigation/)
- [API reference](/api/)
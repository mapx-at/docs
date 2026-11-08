# SDK mapx-gl

A small add-on for MapLibre GL JS: map factory, typed helpers for every API, route planner, layer switcher, parcel and departures popups.

`mapx-gl` is a small add-on (ESM, TypeScript) for MapLibre GL JS. It sets the style URLs, registers the `pmtiles://` protocol, wraps the [APIs](/api/) in typed functions and adds ready-made controls. Peer dependencies: `maplibre-gl@{V.maplibre}` and `pmtiles@{V.pmtiles}`.

## Install

Load the bundle from `https://map.mapx.at/sdk/v1/` (CORS is open) or download the files and serve them with your page:

| File | Use |
|---|---|
| `mapx-gl.bundle.js` | Everything included, load as `<script type="module">` |
| `mapx-gl.js` | Without peers, for projects with their own build |
| `mapx-gl.css` | Styles for the controls, load after `maplibre-gl.css` |
| `mapx-gl.d.ts` | TypeScript types |

```html
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
<link rel="stylesheet" href="https://map.mapx.at/sdk/v1/mapx-gl.css">
<div id="map" style="height:100vh"></div>
<script type="module">
  import mapx from "https://map.mapx.at/sdk/v1/mapx-gl.bundle.js";
  const map = mapx.map({ accessToken: "YOUR_TOKEN", container: "map", style: "streets", center: [15.33, 48.23], zoom: 12 });
  map.addControl(new mapx.SearchControl(), "top-left");
  map.addControl(new mapx.RouteControl(), "top-right");
  map.addControl(new mapx.LayerControl(), "top-right");
</script>
```

A complete example is at [map.mapx.at/sdk/v1/example.html](https://map.mapx.at/sdk/v1/example.html).

## Map

`mapx.map(options)` returns a MapLibre map. `accessToken` is your public token ([Access and limits](/start/access-and-limits/)); it applies to the map and to all helpers. Without a map, call `mapx.setAccessToken("YOUR_TOKEN")` once. `style` is a short name (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`) that becomes `https://api.mapx.at/v1/styles/{id}`, or any style URL or object. `terrain: true` turns on 3D terrain.

## What is in the box

| Part | Content | Page |
|---|---|---|
| Helpers | `geocode`, `reverse`, `searchSuggest`, `searchRetrieve`, `directions`, `transitDirections`, `isochrone`, `matrix`, `optimization`, `parcel`, `parcelDossier`, `tilequery`, `elevation`, `elevationProfile`, `trafficIncidents`, `stops`, `departures`, `weatherCurrent`, `weatherForecast`, `staticImageUrl` | [SDK helpers](/guides/sdk/helpers/) |
| Controls | `SearchControl`, `StyleSwitcherControl`, `MeasureControl`, `ElevationProfileControl`, `Terrain3DControl`, `RouteControl`, `LayerControl` | [Controls and popups](/guides/sdk/controls/) |
| Popups | `ParcelPopup`, `DeparturesPopup` | [Controls and popups](/guides/sdk/controls/) |
| Errors | `MapxError` with `code`, `param`, `allowed`, `retryAfter` | [Errors and retries](/guides/sdk/errors/) |
| Types | Every option and response type | [SDK reference](/reference/sdk/) |

```js
const r = await mapx.directions("driving-truck", [[16.3725, 48.2085], [15.3294, 48.2272]], { truck: { weight: 40, height: 4 } });
const h = await mapx.elevation([12.694, 47.074]); // 3726.3
```

## Next

- [Examples](/examples/) with the SDK: [route planner and layers](/examples/sdk-controls/), [departures](/examples/departures/)
- [Maps](/guides/maps/) and [Navigation](/guides/navigation/)
- [API reference](/api/)
# SDK mapx-gl for MapLibre

Source: https://docs.mapx.at/guides/sdk/

> A small add-on for MapLibre GL JS: map factory, typed helpers for every API, route planner, layer switcher, parcel and departures popups.

`mapx-gl` is a small add-on (ESM, TypeScript) for MapLibre GL JS. It sets the style URLs, registers the `pmtiles://` protocol, wraps the [APIs](https://docs.mapx.at/api/index.md) in typed functions and adds ready-made controls. Peer dependencies: `maplibre-gl@6.18.0` and `pmtiles@{V.pmtiles}`.

## Install

Load the bundle from `https://map.mapx.at/sdk/v1/` (CORS is open) or download the files and serve them with your page:

| File | Use |
|---|---|
| `mapx-gl.bundle.js` | Everything included, load as `<script type="module">` |
| `maplibre-gl-worker.js` | Worker of MapLibre GL JS {V.maplibre}, loaded by the bundle itself; when downloading, put it in the same directory as `mapx-gl.bundle.js` |
| `mapx-gl.js` | Without peers, for projects with their own build |
| `mapx-gl.css` | Styles for the controls, load after `maplibre-gl.css` |
| `mapx-gl.d.ts` | TypeScript types |

```html
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.css">
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

`mapx.map(options)` returns a MapLibre map. `accessToken` is your public token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); it applies to the map and to all helpers. Without a map, call `mapx.setAccessToken("YOUR_TOKEN")` once. `style` is a short name (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`) that becomes `https://api.mapx.at/v1/styles/{id}`, or any style URL or object. `terrain: true` turns on 3D terrain.

## What is in the box

| Part | Content | Page |
|---|---|---|
| Helpers | `geocode`, `reverse`, `searchSuggest`, `searchRetrieve`, `directions`, `transitDirections`, `isochrone`, `matrix`, `optimization`, `parcel`, `parcelDossier`, `tilequery`, `elevation`, `elevationProfile`, `trafficIncidents`, `stops`, `departures`, `weatherCurrent`, `weatherForecast`, `staticImageUrl` | [SDK helpers](https://docs.mapx.at/guides/sdk/helpers.md) |
| Controls | `SearchControl`, `StyleSwitcherControl`, `MeasureControl`, `ElevationProfileControl`, `Terrain3DControl`, `RouteControl`, `LayerControl` | [Controls and popups](https://docs.mapx.at/guides/sdk/controls.md) |
| Popups | `ParcelPopup`, `DeparturesPopup` | [Controls and popups](https://docs.mapx.at/guides/sdk/controls.md) |
| Errors | `MapxError` with `code`, `param`, `allowed`, `retryAfter` | [Errors and retries](https://docs.mapx.at/guides/sdk/errors.md) |
| Types | Every option and response type | [SDK reference](https://docs.mapx.at/reference/sdk/index.md) |

```js
const r = await mapx.directions("driving-truck", [[16.3725, 48.2085], [15.3294, 48.2272]], { truck: { weight: 40, height: 4 } });
const h = await mapx.elevation([12.694, 47.074]); // 3726.3
```

## Next

- [Examples](https://docs.mapx.at/examples/index.md) with the SDK: [route planner and layers](https://docs.mapx.at/examples/sdk-controls.md), [departures](https://docs.mapx.at/examples/departures.md)
- [Maps](https://docs.mapx.at/guides/maps.md) and [Navigation](https://docs.mapx.at/guides/navigation.md)
- [API reference](https://docs.mapx.at/api/index.md)

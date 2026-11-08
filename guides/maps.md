# Maps

Show a mapx map, switch styles, add tilesets and overlays.

mapx delivers maps as ready-made styles and as tiles. Use a style to get a complete map in one line; use tilesets when you build your own style.

**Before you start:** a page with MapLibre GL JS, see [First map](/start/first-map/). You need a free token ([Access and limits](/start/access-and-limits/)); the snippets show `YOUR_TOKEN` only where it matters. The map options below assume the `transformRequest` from First map, which adds the token to every request to `mapx.at`.

## Use a style

| Style | Look |
|---|---|
| `streets` | Base map, roads, places, buildings |
| `streets-dawn`, `streets-dusk`, `streets-night` | The same map in other light |
| `satellite` | Orthophoto |
| `satellite-streets` | Orthophoto with roads and labels |
| `outdoors` | Contours, hillshade, trails |

```js
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  transformRequest: withToken, // from First map: adds the token to every mapx.at request
  center: [12.694, 47.074], zoom: 12, // Grossglockner
});
```

Switching style at runtime replaces layers and sources: call `map.setStyle(url)`. See [Switch the basemap](/examples/switch-basemap/).

## Add tilesets to your own style

Every tileset has a TileJSON address `https://tiles.mapx.at/v1/tiles/{tileset}.json`. MapLibre reads zoom range, bounds and attribution from it.

| Tileset | Type | Content |
|---|---|---|
| `streets` | vector | Roads, rail, water, boundaries, places, buildings |
| `cadastre` | vector | Parcel boundaries (z16) |
| `contours` | vector | Contour lines |
| `terrain-rgb`, `dsm` | raster-dem | Terrain and surface height |
| `orthophoto` | raster | Aerial photo, up to z18 |

```js
map.on("load", () => {
  map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
  map.addLayer({ id: "parcels", type: "line", source: "cadastre", "source-layer": "parcel_l",
                 paint: { "line-color": "#e11d48", "line-width": 1 } });
});
```

Layer names and fields: [Tilesets](/tilesets/). Add your own data the same way with a `geojson` source.

## Weather and traffic overlays

Raster weather tiles have a time dimension; the TileJSON of `temperature`, `precipitation`, `wind`, `humidity`, `cloud` or `pressure` lists the available times. The same URL scheme serves the air quality layers `aqi` (daily index), `no2`, `o3`, `pm10`, `pm25` and `dust` (Saharan dust), see [Air Quality API](/api/air-quality/). Official weather warnings are GeoJSON (`/v1/weather/warnings`, add it as a `geojson` source and colour by `level`).

```js
map.addSource("temp", { type: "raster", url: "https://api.mapx.at/v1/weather/tiles/temperature.json", tileSize: 256 });
map.addLayer({ id: "temp", type: "raster", source: "temp", paint: { "raster-opacity": 0.6 } });
```

The traffic layer is a vector source, see [Traffic](/guides/traffic/). Example: [Weather overlay](/examples/weather-overlay/).

## Other clients

The style and TileJSON formats work in MapLibre, Mapbox GL JS clients that accept custom URLs, Leaflet with a vector plugin, OpenLayers and QGIS. For clients without vector support, [Static Tiles](/api/styles/) deliver the rendered style as raster tiles:

```
https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=YOUR_TOKEN
```

## Offline and bulk use

Each tileset is also one PMTiles file. Download it once instead of fetching tiles: [Self-host PMTiles](/guides/self-host-pmtiles/).

> Show the data source on the map. MapLibre does this automatically when you load a style or TileJSON, see [Attribution](/reference/attribution/).

## Next

- [Style the basemap](/guides/style-the-basemap/)
- [Tilesets](/tilesets/)
- [Styles API](/api/styles/) and [Tiles API](/api/tiles/)
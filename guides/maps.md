# Maps guide: vector tiles and styles

Source: https://docs.mapx.at/guides/maps/

> Show a mapx vector map in MapLibre GL JS, switch styles, add tilesets and overlays, including weather and traffic layers.

mapx delivers maps as ready-made styles and as tiles. Use a style to get a complete map in one line; use tilesets when you build your own style.

**Before you start:** a page with MapLibre GL JS, see [First map](https://docs.mapx.at/start/first-map.md). You need a free token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); the snippets show `YOUR_TOKEN` only where it matters. The style example shows the handover with `transformRequest`; the later snippets assume it.

## Use a style

| Style | Look |
|---|---|
| `streets` | Base map, roads, places, buildings |
| `streets-dawn`, `streets-dusk`, `streets-night` | The same map in other light |
| `satellite` | Orthophoto |
| `satellite-streets` | Orthophoto with roads and labels |
| `outdoors` | Contours, hillshade, trails |

```js
const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  // adds the token to every request to mapx.at
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
  center: [12.694, 47.074], zoom: 12, // Grossglockner
});
```

Switching style at runtime replaces layers and sources: call `map.setStyle(url)`. See [Switch the basemap](https://docs.mapx.at/examples/switch-basemap.md).

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

Layer names and fields: [Tilesets](https://docs.mapx.at/tilesets/index.md). Add your own data the same way with a `geojson` source.

## Weather and traffic overlays

Raster weather tiles have a time dimension; the TileJSON of `temperature`, `precipitation`, `wind`, `humidity`, `cloud` or `pressure` lists the available times. The same URL scheme serves the air quality layers `aqi` (daily index), `no2`, `o3`, `pm10`, `pm25` and `dust` (Saharan dust), see [Air Quality API](https://docs.mapx.at/api/air-quality.md).

```js
map.addSource("temp", { type: "raster", url: "https://api.mapx.at/v1/weather/tiles/temperature.json", tileSize: 256 });
map.addLayer({ id: "temp", type: "raster", source: "temp", paint: { "raster-opacity": 0.6 } });
```

The traffic layer is a vector source, see [Traffic](https://docs.mapx.at/guides/traffic.md). Example: [Weather overlay](https://docs.mapx.at/examples/weather-overlay.md).

## Other clients

The style and TileJSON formats work in MapLibre, Mapbox GL JS clients that accept custom URLs and [QGIS](https://docs.mapx.at/guides/qgis.md). For clients without vector support, [Static Tiles](https://docs.mapx.at/api/styles.md) deliver the rendered style as raster tiles; this is how [OpenLayers](https://docs.mapx.at/guides/openlayers.md) and [Leaflet](https://docs.mapx.at/guides/leaflet.md) add the basemap:

```
https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=YOUR_TOKEN
```

## Offline and bulk use

Tiles and tilesets are available only through the API: style, TileJSON or `{z}/{x}/{y}`. mapx does not offer whole files for download. For many requests, cache tiles in the client, see [Access and limits](https://docs.mapx.at/start/access-and-limits.md).

> Show the data source on the map. MapLibre does this automatically when you load a style or TileJSON, see [Attribution](https://docs.mapx.at/reference/attribution.md).

## Next

- [Style the basemap](https://docs.mapx.at/guides/style-the-basemap.md)
- [Tilesets](https://docs.mapx.at/tilesets/index.md)
- [Styles API](https://docs.mapx.at/api/styles.md) and [Tiles API](https://docs.mapx.at/api/tiles.md)

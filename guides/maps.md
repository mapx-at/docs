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
| `light`, `dark` | Calm, almost monochrome base map (light or dark) for your own data |
| `outdoors` | Contours, hillshade, trails |
| `outdoors-winter` | Like `outdoors` in snow colours, ski areas and cable cars/lifts highlighted |
| `relief` | Pure relief map: elevation tints from green valleys through yellow, ochre and brown to grey and white at the peaks, strong hillshade, lakes and rivers, a few names (places, peaks with height, water); no roads or buildings |
| `relief-contours` | Like `relief`, with contour lines and their heights |
| `relief-light`, `relief-dark` | Grey hillshade only on a light or dark background, very restrained, a backdrop for your own data |

The relief styles need MapLibre GL JS 5.6 or later (layer type `color-relief`) and exist only for maps in the browser, not as static images or static tiles. They show Austria only (terrain model BEV ALS-DTM 1 m, z5 to z16); the area outside stays plain.

### 3D styles

| Style | Content |
|---|---|
| `outdoors-3d`, `outdoors-winter-3d`, `relief-3d` | `outdoors`, `outdoors-winter` or `relief` draped over the real terrain model, with sky and horizon |
| `satellite-3d`, `satellite-streets-3d` | Aerial imagery over the terrain, plain or with roads and names |
| `streets-3d`, `streets-night-3d` | City map with extruded buildings (heights from the BEV building model) and scene lighting, by day or by night |
| `light-3d`, `dark-3d` | Quiet light or dark base map with grey or anthracite buildings, for your own data |

The 3D styles turn on the sky (`sky`) and, except for the city styles, the terrain (`terrain` on the terrain model `terrain-rgb`) in the style and come with a tilted start view (`pitch` 60, `bearing`). MapLibre takes pitch and bearing from the style only when the map is created without a camera position; when you pass `center` and `zoom`, also pass `pitch` and `bearing`, for example `pitch: 60, bearing: -20`. Buildings appear as bodies from about zoom 15 (heights are in the tiles from z15; with a tilted map the background still shows them flat). Terrain covers Austria only. The 3D styles exist only for maps in the browser (MapLibre GL JS), not as static images or static tiles; use the style of the same name without `-3d` there.

```js
const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  // adds the token to every request to mapx.at
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
  center: [12.694, 47.074], zoom: 12, // Grossglockner
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```

Switching style at runtime replaces layers and sources: call `map.setStyle(url)`. See [Switch the basemap](https://docs.mapx.at/examples/switch-basemap.md).

## Add tilesets to your own style

Every tileset has a TileJSON address `https://tiles.mapx.at/v1/tiles/{tileset}.json`. MapLibre reads zoom range, bounds and attribution from it.

| Tileset | Type | Content |
|---|---|---|
| `streets` | vector | Roads, rail, water, boundaries, places, buildings |
| `contours` | vector | Contour lines |
| `terrain-rgb`, `dsm` | raster-dem | Terrain and surface height |
| `orthophoto` | raster | Aerial photo, up to z18 |

```js
map.on("load", () => {
  map.addSource("contours", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" });
  map.addLayer({ id: "contours", type: "line", source: "contours", "source-layer": "contour_l",
                 paint: { "line-color": "#92400e", "line-width": 0.6 } });
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

> Show the data source on the map. MapLibre does this automatically when you load a style or TileJSON, with `mapx.at` (link) as the first entry; please do not remove it; the code above collapses it behind the "i" button, see [Attribution](https://docs.mapx.at/reference/attribution.md).

## Next

- [Style the basemap](https://docs.mapx.at/guides/style-the-basemap.md)
- [Tilesets](https://docs.mapx.at/tilesets/index.md)
- [Styles API](https://docs.mapx.at/api/styles.md) and [Tiles API](https://docs.mapx.at/api/tiles.md)

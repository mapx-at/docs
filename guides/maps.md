# Maps guide: vector tiles and styles

Source: https://docs.mapx.at/guides/maps/

> Show a mapx vector map in MapLibre GL JS, switch styles, add tilesets and overlays, including weather and traffic layers.

mapx delivers maps as ready-made styles and as tiles. Use a style to get a complete map in one line; use tilesets when you build your own style.

**Before you start:** a page with MapLibre GL JS, see [First map](https://docs.mapx.at/start/first-map.md). You need a free token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); the snippets show `YOUR_TOKEN` only where it matters. The style example shows the handover with `transformRequest`; the later snippets assume it.

**PMTiles protocol:** the mapx styles load their tiles from PMTiles files. The page must therefore include `pmtiles.js` and call `maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile)` once; otherwise the map stays empty (console: `URL scheme "pmtiles" is not supported`). All snippets below assume it. Your own styles that only use TileJSON sources (`https://tiles.mapx.at/v1/tiles/{tileset}.json`) do not need it.

## Use a style

| Style | Look |
|---|---|
| `streets` | Base map, roads, places, buildings |
| `streets-dawn`, `streets-dusk`, `streets-night` | The same map in other light |
| `satellite` | Orthophoto |
| `satellite-current` | Current satellite image (Sentinel-2) instead of the orthophoto |
| `satellite-streets` | Orthophoto with roads and labels |
| `light`, `dark` | Calm, almost monochrome base map (light or dark) for your own data |
| `outdoors` | Contours, hillshade, trails |
| `outdoors-winter` | Like `outdoors` in snow colours, ski areas and cable cars/lifts highlighted |
| `relief` | Pure relief map: elevation tints from green valleys through yellow, ochre and brown to grey and white at the peaks, strong hillshade, lakes and rivers, a few names (places, peaks with height, water); no roads or buildings |
| `relief-contours` | Like `relief`, with contour lines and their heights |
| `relief-light`, `relief-dark` | Grey hillshade only on a light or dark background, very restrained, a backdrop for your own data |
| `traffic-day`, `traffic-night` | Map with the current traffic reports and closures (day or night), see [Traffic](https://docs.mapx.at/guides/traffic.md) |

The relief styles need MapLibre GL JS 5.6 or later (layer type `color-relief`) and exist only for maps in the browser. They show Austria only (terrain model BEV ALS-DTM 1 m, z5 to z16); the area outside stays plain.

**As an image:** [Static images](https://docs.mapx.at/guides/static-images.md) and [Static Tiles](https://docs.mapx.at/api/styles.md) render `streets` (with the time-of-day variants), `light`, `dark`, `satellite`, `satellite-streets`, `outdoors` (the full vector map) and `outdoors-winter`. Not available as an image are `traffic-day` and `traffic-night` (answer `404` with a hint to use `streets` or `streets-night`), `satellite-current`, the relief styles and all `-3d` styles.

### 3D styles

| Style | Content |
|---|---|
| `outdoors-3d`, `outdoors-winter-3d`, `relief-3d` | `outdoors`, `outdoors-winter` or `relief` draped over the real terrain model, with sky and horizon |
| `satellite-3d`, `satellite-streets-3d` | Aerial imagery over the terrain, plain or with roads and names |
| `streets-3d`, `streets-night-3d` | City map with extruded buildings (heights from the BEV building model) and scene lighting, by day or by night |
| `light-3d`, `dark-3d` | Quiet light or dark base map with grey or anthracite buildings, for your own data |

The 3D styles turn on the sky (`sky`) and the terrain (`terrain` on the terrain model `terrain-rgb`, without exaggeration in the city styles) in the style and come with a tilted start view (`pitch` 72, city 68, plus `bearing`). MapLibre takes pitch and bearing from the style only when the map is created without a camera position; when you pass `center` and `zoom`, also pass `pitch` and `bearing`. By default MapLibre allows at most 60 degrees of pitch, which usually keeps the horizon out of view. To see the sky, set `maxPitch` and `pitch` yourself:

```js
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile); // once per page, see above
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets-3d",
  // transformRequest as in the snippet further down
  center: [16.3731, 48.2085], zoom: 15,
  maxPitch: 78, pitch: 74, bearing: -20, // the sky comes into view from about 72 degrees of pitch
});
// City styles: buildings up to the horizon (set again after every style load)
map.on("style.load", () => map.setSourceTileLodParams(3, 6));
```

At a steep pitch MapLibre loads tiles of lower zoom levels for the distance. Below zoom 12 they contain no buildings, so the building bodies stop in the background. `setSourceTileLodParams(3, 6)` limits the number of zoom levels in view to three, so the bodies reach the horizon; the map loads a few more tiles in return. The terrain and aerial styles do not need this.

Keep `maxPitch` at 78 or below in the city styles: at a steeper pitch MapLibre cuts the terrain off at the horizon but still draws the buildings on hills behind it, so they appear to float.

Buildings rise as bodies between zoom 12 and 13 and then also stand in the distance of a tilted map (merged buildings with an average height, from zoom 15 every building with its own height). Heights come from the BEV building model, missing ones from the elevation models (surface minus terrain); tall towers are separate parts (`building_part_a`) on top of the buildings. Terrain covers Austria only. The 3D styles exist only for maps in the browser (MapLibre GL JS), not as static images or static tiles; use the style without `-3d` there (there is no image for `relief-3d` either).

```js
const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile); // the styles load PMTiles
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  // adds the token to every request to mapx.at
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
  center: [12.694, 47.074], zoom: 12, // Grossglockner
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```

Switching style at runtime replaces layers and sources: call `map.setStyle(url)`. Add your own sources and layers again afterwards (for example in the `style.load` event). See [Switch the basemap](https://docs.mapx.at/examples/switch-basemap.md).

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
  map.addSource("mx-contours", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" });
  map.addLayer({ id: "mx-contours", type: "line", source: "mx-contours", "source-layer": "contour_l",
                 paint: { "line-color": "#92400e", "line-width": 0.6 } });
});
```

The mapx styles already use the source names `streets`, `contours`, `terrain-rgb`, `dsm`, `orthophoto`, `traffic` and others. Pick your own name when you add a source to a mapx style (here `mx-contours`); `outdoors` already shows contour lines. Layer names and fields: [Tilesets](https://docs.mapx.at/tilesets/index.md). Add your own data the same way with a `geojson` source; a complete example is [GeoJSON overlay](https://docs.mapx.at/examples/geojson-overlay.md).

## Weather and traffic overlays

Raster weather tiles have a time dimension; the TileJSON of `temperature`, `precipitation`, `wind`, `humidity`, `cloud` or `pressure` lists the available times in the field `mapx:time.times`. The tile URL contains `/now/` by default; to choose another time, put an entry of that list in place of `now` (template in `mapx:time.template`). The same URL scheme serves the air quality layers `aqi` (daily index), `no2`, `o3`, `pm10`, `pm25` and `dust` (Saharan dust), see [Air Quality API](https://docs.mapx.at/api/air-quality.md).

```js
map.addSource("temp", { type: "raster", url: "https://api.mapx.at/v1/weather/tiles/temperature.json", tileSize: 256 });
map.addLayer({ id: "temp", type: "raster", source: "temp", paint: { "raster-opacity": 0.6 } });
```

The traffic layer is a vector source, see [Traffic](https://docs.mapx.at/guides/traffic.md). Example: [Weather overlay](https://docs.mapx.at/examples/weather-overlay.md).

## Other clients

The TileJSON format works in MapLibre and [QGIS](https://docs.mapx.at/guides/qgis.md). The styles themselves load PMTiles files (`pmtiles://`) and therefore run only in clients that know this protocol, such as MapLibre with `pmtiles.js`. For clients without vector support, [Static Tiles](https://docs.mapx.at/api/styles.md) deliver the rendered style as raster tiles; this is how [OpenLayers](https://docs.mapx.at/guides/openlayers.md) and [Leaflet](https://docs.mapx.at/guides/leaflet.md) add the basemap:

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

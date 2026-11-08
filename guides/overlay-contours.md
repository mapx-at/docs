# Overlay contour lines


**Goal:** Place contour lines of suitable density over aerial imagery, draw index lines more strongly and label them with their elevation.

**Prerequisites:** a browser, a static web server and a free public token ([Access and limits](/start/access-and-limits/)). Labels use the mapx fonts, see [Style the basemap](/guides/style-the-basemap/#labels-and-fonts).

The [contour lines](/tilesets/contours/) tileset has a layer `contour_l` with the fields `ele` (elevation in m), `index` (1 = index line) and `lvl` (generalisation level 10, 12, 14, 16). Lines exist from z10.


1. **Page with background.** The orthophoto as base, the contours as a vector source:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
   <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
   <style>html, body, #map { margin: 0; height: 100%; }</style>
   <div id="map"></div>
   <script>
     const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
     // adds the token to every request to mapx.at (TileJSON, tiles, fonts)
     const withToken = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url });
     const sources = {
       ortho: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 },
       contours: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" }
     };
   ```

2. **Pick the level.** The tiles contain several levels with their own line interval. To show only the matching level per zoom, filter by `lvl`:

   | Zoom | `lvl` | Interval | Index line |
   |---|---|---|---|
   | 10–11 | 10 | 100 m | 500 m |
   | 12–13 | 12 | 50 m | 200 m |
   | 14–15 | 14 | 20 m | 100 m |
   | from 16 | 16 | 10 m | 50 m |

   One layer per level, limited with `minzoom` and `maxzoom`:

   ```js
   const levels = [
     { lvl: 10, min: 10, max: 12 },
     { lvl: 12, min: 12, max: 14 },
     { lvl: 14, min: 14, max: 16 },
     { lvl: 16, min: 16, max: 24 }
   ];
   const lineLayers = levels.flatMap(({ lvl, min, max }) => [
     { id: `line-${lvl}`, type: "line", source: "contours", "source-layer": "contour_l",
       minzoom: min, maxzoom: max,
       filter: ["all", ["==", ["get", "lvl"], lvl], ["!=", ["get", "index"], 1]],
       paint: { "line-color": "#ffd9a0", "line-width": 0.6, "line-opacity": 0.8 } },
     { id: `index-${lvl}`, type: "line", source: "contours", "source-layer": "contour_l",
       minzoom: min, maxzoom: max,
       filter: ["all", ["==", ["get", "lvl"], lvl], ["==", ["get", "index"], 1]],
       paint: { "line-color": "#ffb347", "line-width": 1.4 } }
   ]);
   ```

3. **Elevation labels.** Only index lines carry a number. `symbol-placement: "line"` runs the text along the line:

   ```js
   const labels = {
     id: "ele-text", type: "symbol", source: "contours", "source-layer": "contour_l", minzoom: 12,
     filter: ["==", ["get", "index"], 1],
     layout: {
       "symbol-placement": "line",
       "text-field": ["concat", ["to-string", ["get", "ele"]], " m"],
       "text-font": ["Open Sans Semibold"],
       "text-size": 11
     },
     paint: { "text-color": "#fff", "text-halo-color": "#7a4a00", "text-halo-width": 1.5 }
   };
   ```

   `text-font` must be a font that `/v1/fonts` lists (`GET /v1/fonts`), for example `Open Sans Semibold` or `Noto Sans Regular`.

4. **Start the map.**

   ```js
     new maplibregl.Map({
       container: "map",
       transformRequest: withToken,
       center: [12.694, 47.074], zoom: 14,
       style: {
         version: 8,
         glyphs: "https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf",
         sources: sources,
         layers: [{ id: "ortho", type: "raster", source: "ortho" }, ...lineLayers, labels]
       }
     });
   </script>
   ```


## Result

Fine contour lines lie over the aerial imagery, their interval follows the zoom (100 m to 10 m). Index lines are strong and carry the elevation in metres.

For a map instead of imagery, replace the orthophoto layer with the layers from [Style the basemap](/guides/style-the-basemap/); put the contours on top.

> Without a filter on `lvl`, several levels overlap when the map overzooms. Lines end below z10; for smaller scales the relief stays visible through [hillshading](/guides/3d-terrain/).


## Next steps

- Fields and levels in detail: [Contour lines](/tilesets/contours/).
- Three-dimensional terrain: [3D terrain](/guides/3d-terrain/).
- Credits: [Attribution](/reference/attribution/).


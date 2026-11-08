# Your own map from the basemap


**Goal:** Build your own, consistently coloured map from the vector tiles of the [basemap](/tilesets/streets/). mapx provides the data, you decide how it looks.

**Prerequisites:** a browser, a static web server and a free public token ([Access and limits](/start/access-and-limits/)). Basic knowledge of MapLibre styles (sources and layers).

The basemap has 18 layers. This example uses five: `landuse_a` (land use), `water_a` (lakes), `road_l` (roads), `building_a` (buildings) and `boundary_l` (boundaries). All layers and fields are in the [layer table](/tilesets/streets/).


1. **Page and source.** The source is the TileJSON link; zoom range and attribution come with it:

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
     const source = { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" };
   ```

2. **Background and land use.** Every polygon in `landuse_a` has a `kind` field (for example `forest`, `farmland`, `vineyard`, `alpine_meadow`, `glacier`, `rock`, `residential`). One colour per group via a `match` expression:

   ```js
   const landuse = {
     id: "landuse", type: "fill", source: "streets", "source-layer": "landuse_a",
     paint: { "fill-color": ["match", ["get", "kind"],
       "forest", "#bcd9b0",
       ["farmland", "vineyard"], "#efe9c8",
       "alpine_meadow", "#d9e8c4",
       "glacier", "#f4f8fb",
       "rock", "#d8d4cf",
       ["residential", "garden"], "#ebe5de",
       "#e6e9dc"] }
   };
   ```

3. **Water.** Lakes and ponds from `water_a` as polygons:

   ```js
   const water = {
     id: "water", type: "fill", source: "streets", "source-layer": "water_a",
     paint: { "fill-color": "#a9cde8" }
   };
   ```

4. **Roads.** `road_l` has `kind`: `highway`, `major_road`, `minor_road`, `service`, `track`, `path`. Let widths grow with zoom, draw paths thin and dashed:

   ```js
   const roads = {
     id: "roads", type: "line", source: "streets", "source-layer": "road_l",
     filter: ["in", ["get", "kind"], ["literal", ["highway", "major_road", "minor_road", "service"]]],
     layout: { "line-cap": "round", "line-join": "round" },
     paint: {
       "line-color": ["match", ["get", "kind"], "highway", "#e8a35c", "major_road", "#f0c27a", "#ffffff"],
       "line-width": ["interpolate", ["exponential", 1.5], ["zoom"], 8, 0.5, 14, 3, 16, 8]
     }
   };
   const trails = {
     id: "trails", type: "line", source: "streets", "source-layer": "road_l", minzoom: 13,
     filter: ["in", ["get", "kind"], ["literal", ["track", "path"]]],
     paint: { "line-color": "#a07a5a", "line-width": 1, "line-dasharray": [2, 1.5] }
   };
   ```

5. **Buildings.** `building_a` starts at z12 (merged there). From z15 each building has its own outline:

   ```js
   const buildings = {
     id: "buildings", type: "fill", source: "streets", "source-layer": "building_a", minzoom: 13,
     paint: { "fill-color": "#cfc7bd", "fill-outline-color": "#a89f94" }
   };
   ```

6. **Boundaries.** `boundary_l` carries `kind` (`country`, `state`, `district`, `municipality`). Strong lines for country and state, subtle for municipalities:

   ```js
   const boundaries = [
     { id: "b-country", type: "line", source: "streets", "source-layer": "boundary_l",
       filter: ["==", ["get", "kind"], "country"],
       paint: { "line-color": "#7a4f8f", "line-width": 1.5 } },
     { id: "b-state", type: "line", source: "streets", "source-layer": "boundary_l",
       filter: ["==", ["get", "kind"], "state"],
       paint: { "line-color": "#9d86a8", "line-width": 1, "line-dasharray": [4, 2] } },
     { id: "b-municipality", type: "line", source: "streets", "source-layer": "boundary_l", minzoom: 10,
       filter: ["==", ["get", "kind"], "municipality"],
       paint: { "line-color": "#b9b0be", "line-width": 0.6, "line-dasharray": [2, 2] } }
   ];
   ```

7. **Assemble the map.** Order from bottom to top: land use, water, roads, buildings, boundaries.

   ```js
     new maplibregl.Map({
       container: "map",
       transformRequest: withToken,
       center: [15.334, 48.227], zoom: 14,
       style: {
         version: 8,
         sources: { streets: source },
         layers: [
           { id: "ground", type: "background", paint: { "background-color": "#f2efe9" } },
           landuse, water, roads, trails, buildings, ...boundaries
         ]
       }
     });
   </script>
   ```


## Result

A map in your own look: areas by land use, lakes, road network with trails, buildings and administrative boundaries. All data comes from a single tileset.

## Labels and fonts

For labels (`place_p`, `water_label_p`, `housenumber_p`, road names from `road_l` via the field `name`) the style needs a `glyphs` entry. mapx serves fonts at `/v1/fonts/{fontstack}/{range}.pbf` (list: `GET /v1/fonts`, for example `Open Sans Semibold`, `Noto Sans Regular`). The `withToken` function from step 1 adds the token to these requests too:

```js
style: {
  version: 8,
  glyphs: "https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf",
  sources: { streets: source },
  layers: [
    // ...
    { id: "places", type: "symbol", source: "streets", "source-layer": "place_p",
      filter: ["in", ["get", "kind"], ["literal", ["city", "town", "village"]]],
      layout: { "text-field": ["get", "name"], "text-font": ["Open Sans Semibold"], "text-size": 13 },
      paint: { "text-color": "#333", "text-halo-color": "#fff", "text-halo-width": 1.5 } }
  ]
}
```

`text-font` must be a font that `GET /v1/fonts` lists.

## Next steps

- All layers, fields and values: [Basemap](/tilesets/streets/).
- Overlay contour lines and labels: [Overlay contour lines](/guides/overlay-contours/).
- A map for print or as an offline file: [Self-host PMTiles](/guides/self-host-pmtiles/).


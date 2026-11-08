# 3D terrain with orthophoto

A 3D map with real elevation relief, hillshading and aerial imagery on top, using MapLibre GL JS and a free token.

**Goal:** A map where the orthophoto is draped over real terrain, shaded for depth and tiltable in the browser.

**Prerequisites:** a browser and any static web server (for example `npx serve`). A free API token ([Access and limits](/start/access-and-limits/)).

This guide uses the [DTM](/tilesets/terrain-rgb/) (elevation, type `raster-dem`) and the [orthophoto](/tilesets/orthophoto/) (imagery, type `raster`). Both tilesets are added via [TileJSON](/api/tiles/), so you never need a file name.

<Steps>

1. **Create the page.** A file `index.html` with MapLibre GL JS and a map container:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
   <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
   <style>html, body, #map { margin: 0; height: 100%; }</style>
   <div id="map"></div>
   ```

2. **Declare the sources.** The elevation model as `raster-dem`, the orthophoto as `raster`:

   ```js
   const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
   // adds the token to every request to mapx.at (TileJSON and tiles)
   const withToken = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url });

   const sources = {
     dem:   { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", encoding: "mapbox" },
     ortho: { type: "raster",     url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 }
   };
   ```

3. **Stack the layers.** Imagery at the bottom, hillshade on top with a light touch so the photo stays recognisable:

   ```js
   const layers = [
     { id: "ortho", type: "raster", source: "ortho" },
     { id: "hillshade", type: "hillshade", source: "dem",
       paint: { "hillshade-exaggeration": 0.4, "hillshade-shadow-color": "#000000" } }
   ];
   ```

4. **Enable terrain.** `terrain` lifts all layers to the DTM elevation. `pitch` tilts the view:

   ```html
   <script>
     const map = new maplibregl.Map({
       container: "map",
       transformRequest: withToken,
       center: [12.694, 47.074], zoom: 12, pitch: 65, bearing: 20,
       maxPitch: 80,
       style: {
         version: 8,
         sources: sources,
         layers: layers,
         terrain: { source: "dem", exaggeration: 1.3 }
       }
     });
     map.addControl(new maplibregl.NavigationControl({ visualizePitch: true }));
   </script>
   ```

   The constants from steps 2 and 3 (`TOKEN`, `withToken`, `sources`, `layers`) must appear in the same `<script>` before `new maplibregl.Map`.

5. **Run it.** Run `npx serve` in the folder and open the printed address. Dragging with the right mouse button tilts and rotates the map.

</Steps>

## Result

The aerial imagery lies on the terrain around the Großglockner, slopes are additionally shaded. The orthophoto reaches z18 (20 cm), the DTM z16 (1 m raster); MapLibre scales up automatically beyond that.

> `exaggeration` exaggerates the relief. `1.0` is natural; for flat terrain `1.5` to `2` looks more plastic.

## Next steps

- Read the elevation at a click: `map.queryTerrainElevation(e.lngLat)`, details in [DTM terrain model](/tilesets/terrain-rgb/).
- Contour lines and labels on top: [Overlay contour lines](/guides/overlay-contours/).
- Elevation profile along a line: [Elevation profile](/guides/elevation-profile/).
- Show the credit correctly: [Attribution](/reference/attribution/).
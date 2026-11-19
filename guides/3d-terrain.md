# 3D terrain map with orthophoto

Source: https://docs.mapx.at/guides/3d-terrain/

> A 3D map with real elevation relief, hillshading and aerial imagery on top, using MapLibre GL JS and a free token.

**Goal:** A map where the orthophoto is draped over real terrain, shaded for depth and tiltable in the browser.

**Prerequisites:** a browser and any static web server (for example `npx serve`). A free API token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)).

This guide uses the [DTM](https://docs.mapx.at/tilesets/terrain-rgb.md) (elevation, type `raster-dem`) and the [orthophoto](https://docs.mapx.at/tilesets/orthophoto.md) (imagery, type `raster`). Both tilesets are added via [TileJSON](https://docs.mapx.at/api/tiles.md), so you never need a file name.

1. **Create the page.** A file `index.html` with MapLibre GL JS and a map container:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.css">
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
   <script type="module">
     import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";
     const map = new maplibregl.Map({
       attributionControl: false,
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
     map.addControl(new maplibregl.AttributionControl({ compact: true }));
     map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
     map.addControl(new maplibregl.NavigationControl({ visualizePitch: true }));
   </script>
   ```

   The constants from steps 2 and 3 (`TOKEN`, `withToken`, `sources`, `layers`) must appear in the same `<script>` before `new maplibregl.Map`.

5. **Run it.** Run `npx serve` in the folder and open the printed address. Dragging with the right mouse button tilts and rotates the map.

## Result

The aerial imagery lies on the terrain around the Großglockner, slopes are additionally shaded. The orthophoto reaches z18 (20 cm), the DTM z16 (1 m raster); MapLibre scales up automatically beyond that.

> `exaggeration` exaggerates the relief. `1.0` is natural; for flat terrain `1.5` to `2` looks more plastic.

## Next steps

- Read the elevation at a click: `map.queryTerrainElevation(e.lngLat)`, details in [DTM terrain model](https://docs.mapx.at/tilesets/terrain-rgb.md).
- Contour lines and labels on top: [Overlay contour lines](https://docs.mapx.at/guides/overlay-contours.md).
- Elevation profile along a line: [Elevation profile](https://docs.mapx.at/guides/elevation-profile.md).
- Show the credit correctly: [Attribution](https://docs.mapx.at/reference/attribution.md).

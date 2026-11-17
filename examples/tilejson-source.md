# TileJSON source example

Source: https://docs.mapx.at/examples/tilejson-source/

> Add the mapx orthophoto and contour lines as TileJSON sources without a ready-made style in MapLibre GL JS: runnable example code.

One TileJSON URL per product is enough: MapLibre reads tile URL, zoom range and attribution from it. Here a custom, minimal style.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Use TileJSON: orthophoto and contour lines as sources, without file names and without a style.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

const map = new maplibregl.Map({
  transformRequest,
  container: 'map', center: [11.4, 47.27], zoom: 12.5, // Innsbruck
  style: {
    version: 8,
    sources: {
      ortho: { type: 'raster', url: 'https://api.mapx.at/v1/tiles/orthophoto.json', tileSize: 512 },
      linien: { type: 'vector', url: 'https://api.mapx.at/v1/tiles/contours.json' },
    },
    layers: [
      { id: 'ortho', type: 'raster', source: 'ortho' },
      { id: 'linien', type: 'line', source: 'linien', 'source-layer': 'contour_l', paint: { 'line-color': '#ffd24a', 'line-width': 0.8, 'line-opacity': 0.9 } },
    ],
  },
});
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
````

## APIs used

- [TileJSON](https://docs.mapx.at/api/tiles.md) `orthophoto`, `contours`

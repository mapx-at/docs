# TileJSON source example

Source: https://docs.mapx.at/examples/tilejson-source/

> Add the mapx orthophoto and contour lines as TileJSON sources without a ready-made style in MapLibre GL JS: runnable example code.

One TileJSON URL per product is enough: MapLibre reads tile URL, zoom range and attribution from it. Here a custom, minimal style. OpenLayers does not read TileJSON for vector tiles itself: the OpenLayers version loads it with `fetch` and builds `ol.source.XYZ` (raster) and `ol.source.VectorTile` with `ol.format.MVT` (vector tiles) from it.

## Code

Leaflet shows only the raster part: it has no vector tiles without a plugin, so the contour lines are missing there.

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Use TileJSON: orthophoto and contour lines as sources, without file names and without a style.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

const map = new maplibregl.Map({
  attributionControl: false,
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
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Use TileJSON: orthophoto (raster) and contour lines (vector tiles, MVT) as sources, without file names and without a style.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
// Data sources (CC BY 4.0), must stay visible on the map; this example takes the text from each TileJSON.

const linie = new ol.style.Style({ stroke: new ol.style.Stroke({ color: 'rgba(255, 210, 74, 0.9)', width: 0.8 }) });
const ortho = new ol.layer.Tile(), linien = new ol.layer.VectorTile({ style: linie });
const map = new ol.Map({ target: 'map', layers: [ortho, linien], view: new ol.View({ center: ol.proj.fromLonLat([11.4, 47.27]), zoom: 13.5 }) }); // Innsbruck

const plain = (html) => new DOMParser().parseFromString(html, 'text/html').body.textContent; // attribution as text, not as HTML
async function tilejson(name, sep = '') {
  const r = await fetch(`https://api.mapx.at/v1/tiles/${name}.json`, AUTH);
  if (!r.ok) throw new Error('HTTP ' + r.status);
  const tj = await r.json(); // tile URL, zoom range and attribution
  return { url: tj.tiles[0] + '?access_token=' + TOKEN, minZoom: tj.minzoom, maxZoom: tj.maxzoom, attributions: plain(tj.attribution ?? '') + sep, attributionsCollapsible: false };
}
(async () => {
  ortho.setSource(new ol.source.XYZ({ ...(await tilejson('orthophoto', '; ')), tileSize: 512 }));
  linien.setSource(new ol.source.VectorTile({ ...(await tilejson('contours')), format: new ol.format.MVT({ layers: ['contour_l'] }) }));
})();
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Use TileJSON: the orthophoto as a raster source, tile URL, zoom range and attribution come from the TileJSON.
// Leaflet has no vector tiles without a plugin, so the contour lines of the MapLibre version are missing here.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

const map = L.map('map').setView([47.27, 11.4], 14); // Innsbruck
(async () => {
  const tj = await (await fetch('https://api.mapx.at/v1/tiles/orthophoto.json', AUTH)).json();
  // the attribution text of the TileJSON (CC BY 4.0) must stay visible on the map; it contains HTML, so reduce it to plain text and escape it
  const plain = new DOMParser().parseFromString(tj.attribution ?? '', 'text/html').body.textContent;
  L.tileLayer(tj.tiles[0] + '?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: tj.minzoom + 1, maxNativeZoom: tj.maxzoom + 1, maxZoom: 21, attribution: Object.assign(document.createElement('span'), { textContent: plain }).innerHTML }).addTo(map);
})();
````

Leaflet does not read TileJSON by itself: the Leaflet version fetches it and takes `tiles`, `minzoom`, `maxzoom` and `attribution`, but shows only the orthophoto, as Leaflet cannot draw vector tiles such as the contour lines without a plugin.

## APIs used

- [TileJSON](https://docs.mapx.at/api/tiles.md) `orthophoto`, `contours`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (Leaflet)

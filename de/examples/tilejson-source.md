# TileJSON-Quellen: Beispiel

Source: https://docs.mapx.at/de/examples/tilejson-source/

> Orthophoto und Höhenlinien von mapx als TileJSON-Quellen ohne fertigen Stil in MapLibre GL JS einbinden: lauffähiger Beispielcode zum Kopieren.

Eine TileJSON-URL pro Produkt genügt: MapLibre liest Kachel-URL, Zoombereich und Attribution daraus. Hier ein eigener, minimaler Stil. OpenLayers liest das TileJSON nicht selbst: Die OpenLayers-Fassung lädt es per `fetch` und baut daraus `ol.source.XYZ` (Raster) und `ol.source.VectorTile` mit `ol.format.MVT` (Vektorkacheln).

## Code

Leaflet zeigt nur den Rasterteil: Vektorkacheln kann es ohne Plugin nicht, die Höhenlinien fehlen dort.

### MapLibre GL JS

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
async function tilejson(name) {
  const r = await fetch(`https://api.mapx.at/v1/tiles/${name}.json`, AUTH);
  if (!r.ok) throw new Error('HTTP ' + r.status);
  const tj = await r.json(); // tile URL, zoom range and attribution
  return { url: tj.tiles[0] + '?access_token=' + TOKEN, minZoom: tj.minzoom, maxZoom: tj.maxzoom, attributions: plain(tj.attribution ?? ''), attributionsCollapsible: false };
}
(async () => {
  ortho.setSource(new ol.source.XYZ({ ...(await tilejson('orthophoto')), tileSize: 512 }));
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
  // the attribution text of the TileJSON (CC BY 4.0) must stay visible on the map; escaped, as it is API text, not HTML
  L.tileLayer(tj.tiles[0] + '?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: tj.minzoom + 1, maxNativeZoom: tj.maxzoom + 1, maxZoom: 21, attribution: Object.assign(document.createElement('span'), { textContent: tj.attribution }).innerHTML }).addTo(map);
})();
````

Leaflet liest kein TileJSON von selbst: Die Leaflet-Fassung holt es mit `fetch` und übernimmt `tiles`, `minzoom`, `maxzoom` und `attribution`, zeigt aber nur das Orthophoto, denn Vektorkacheln wie die Höhenlinien kann Leaflet ohne Plugin nicht.

## Verwendete APIs

- [TileJSON](https://docs.mapx.at/de/api/tiles.md) `orthophoto`, `contours`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (Leaflet)

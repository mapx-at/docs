# Switch basemap example

Source: https://docs.mapx.at/examples/switch-basemap/

> Switch between the map, aerial imagery (orthophoto) and terrain with MapLibre GL JS and mapx styles: a runnable basemap switcher example.

Three ready-made styles share position and zoom. `map.setStyle()` swaps the base, the view stays. In OpenLayers, `setSource()` swaps the style's raster tiles and the view stays as well.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Switch the basemap: map, aerial imagery and terrain are three ready-made styles.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STILE = {
  streets: 'https://api.mapx.at/v1/styles/streets',
  satellite: 'https://api.mapx.at/v1/styles/satellite',
  outdoors: 'https://api.mapx.at/v1/styles/outdoors',
};
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: STILE.streets, center: [11.4, 47.27], zoom: 11 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

for (const knopf of document.querySelectorAll('[data-stil]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(STILE[knopf.dataset.stil]); // position and zoom are kept
    for (const k of document.querySelectorAll('[data-stil]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Switch the basemap: map, aerial imagery and terrain are three ready-made styles, here as raster tiles.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = {
  streets: '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
  satellite: 'Orthophoto: <a href="https://www.bev.gv.at">BEV</a>, <a href="https://basemap.at">basemap.at</a>; <a href="https://www.gip.gv.at">GIP.at</a>; <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
  outdoors: '<a href="https://www.bev.gv.at">BEV</a> (incl. ALS-DTM), <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
};
const tiles = (style) => new ol.source.XYZ({ url: `https://api.mapx.at/v1/styles/${style}/tiles/512/{z}/{x}/{y}?access_token=` + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION[style], attributionsCollapsible: false });
const base = new ol.layer.Tile({ source: tiles('streets') });
const map = new ol.Map({ target: 'map', layers: [base], view: new ol.View({ center: ol.proj.fromLonLat([11.4, 47.27]), zoom: 12 }) }); // Innsbruck

for (const knopf of document.querySelectorAll('[data-stil]')) {
  knopf.addEventListener('click', () => {
    base.setSource(tiles(knopf.dataset.stil)); // position and zoom are kept
    for (const k of document.querySelectorAll('[data-stil]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Switch the basemap: map, aerial imagery and terrain are three ready-made styles, here as raster tiles (Static Tiles).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Data sources per style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const CC = '<a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';
const ATTRIBUTION = {
  streets: `<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, ${CC}`,
  satellite: `Orthophoto: <a href="https://www.bev.gv.at">BEV</a>, <a href="https://basemap.at">basemap.at</a>; <a href="https://www.gip.gv.at">GIP.at</a>, ${CC}`,
  outdoors: `<a href="https://www.bev.gv.at">BEV</a> (ALS-DTM), <a href="https://www.gip.gv.at">GIP.at</a>, ${CC}`,
};
const tiles = (style) => L.tileLayer(`https://api.mapx.at/v1/styles/${style}/tiles/512/{z}/{x}/{y}?access_token=${TOKEN}`, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION[style] });

const map = L.map('map').setView([47.27, 11.4], 12); // Innsbruck
let base = tiles('streets').addTo(map);
for (const knopf of document.querySelectorAll('[data-stil]')) {
  knopf.addEventListener('click', () => {
    base.remove(); // position and zoom are kept
    base = tiles(knopf.dataset.stil).addTo(map);
    for (const k of document.querySelectorAll('[data-stil]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
````

In Leaflet the example swaps the tile layer (`L.tileLayer`) together with its attribution, the view stays as well.

## APIs used

- [Styles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{streets|satellite|outdoors}`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

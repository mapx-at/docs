# Time of day and night mode example

Source: https://docs.mapx.at/examples/time-of-day/

> Show the default mapx style in four lighting moods, including a night mode, with MapLibre GL JS: runnable example code to copy.

Morning, day, evening and night are separate palettes of the same style. For a dark UI the night variant is enough. In OpenLayers, `setSource()` swaps the raster tiles of the lighting mood.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Time of day and night mode: the standard styles come in four lighting moods.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STIL = { morgen: 'streets-dawn', tag: 'streets', abend: 'streets-dusk', nacht: 'streets-night' };
const stil = (licht) => `https://api.mapx.at/v1/styles/${STIL[licht]}`;
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: stil('tag'), center: [16.37, 48.21], zoom: 12 }); // Wien
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

for (const knopf of document.querySelectorAll('[data-licht]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(stil(knopf.dataset.licht));
    for (const k of document.querySelectorAll('[data-licht]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Time of day and night mode: the standard style comes in four lighting moods, here as raster tiles.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const STIL = { morgen: 'streets-dawn', tag: 'streets', abend: 'streets-dusk', nacht: 'streets-night' };
const tiles = (style) => new ol.source.XYZ({ url: `https://api.mapx.at/v1/styles/${style}/tiles/512/{z}/{x}/{y}?access_token=` + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false });
const base = new ol.layer.Tile({ source: tiles(STIL.tag) });
const map = new ol.Map({ target: 'map', layers: [base], view: new ol.View({ center: ol.proj.fromLonLat([16.37, 48.21]), zoom: 13 }) }); // Wien

for (const knopf of document.querySelectorAll('[data-licht]')) {
  knopf.addEventListener('click', () => {
    base.setSource(tiles(STIL[knopf.dataset.licht]));
    for (const k of document.querySelectorAll('[data-licht]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Time of day and night mode: the standard styles come in four lighting moods, here as raster tiles (Static Tiles).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Data sources of the style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';
const STIL = { morgen: 'streets-dawn', tag: 'streets', abend: 'streets-dusk', nacht: 'streets-night' };
const url = (licht) => `https://api.mapx.at/v1/styles/${STIL[licht]}/tiles/512/{z}/{x}/{y}?access_token=${TOKEN}`;

const map = L.map('map').setView([48.21, 16.37], 13); // Wien
const base = L.tileLayer(url('tag'), { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
for (const knopf of document.querySelectorAll('[data-licht]')) {
  knopf.addEventListener('click', () => {
    base.setUrl(url(knopf.dataset.licht)); // same layer, new tiles
    for (const k of document.querySelectorAll('[data-licht]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
````

In Leaflet `setUrl()` switches the raster tiles of the variant on the same layer.

## APIs used

- [Styles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{streets-dawn|streets|streets-dusk|streets-night}`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

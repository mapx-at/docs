# Karte in eine HTML-Seite einbetten

Source: https://docs.mapx.at/de/examples/embed-map/

> Eine mapx-Karte mit dem fertigen Stil per MapLibre GL JS in eine beliebige HTML-Seite einbetten: minimales, lauffähiges Beispiel mit öffentlichem Token.

Der kürzeste Weg zu einer Karte: MapLibre GL JS, das pmtiles-Protokoll und die Stil-URL. Zoom, Neigung und Maßstab kommen als Steuerelemente dazu.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Embed a map: the ready-made mapx style, one call.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({
  attributionControl: false,
  transformRequest,
  container: 'map',
  style: 'https://api.mapx.at/v1/styles/streets',
  center: [13.04, 47.80], // Salzburg
  zoom: 12,
});
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
map.addControl(new maplibregl.ScaleControl());
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Embed a map: the mapx style as rendered raster tiles (Static Tiles) in OpenLayers.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Data sources of the style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = new ol.Map({
  target: 'map',
  layers: [new ol.layer.Tile({ source: new ol.source.XYZ({
    url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN,
    tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false,
  }) })],
  view: new ol.View({ center: ol.proj.fromLonLat([13.04, 47.80]), zoom: 13, maxZoom: 21 }), // Salzburg
});
map.addControl(new ol.control.ScaleLine());
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Embed a map: the mapx style as rendered raster tiles (Static Tiles) in Leaflet.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Data sources of the style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.80, 13.04], 13); // Salzburg
// 512 px tiles: zoomOffset -1 keeps the map scale of standard 256 px tiles
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, {
  tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION,
}).addTo(map);
L.control.scale({ imperial: false }).addTo(map);
````

## Verwendete APIs

- [Styles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/streets`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

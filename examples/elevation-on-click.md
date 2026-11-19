# Elevation on click example

Source: https://docs.mapx.at/examples/elevation-on-click/

> A map click queries the elevation at a point, either terrain height or surface height including buildings, with the mapx elevation API.

The elevation API returns GeoJSON with `elevation` in metres. The DTM shows the terrain, the DSM buildings and vegetation.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Query elevation: a click asks /v1/elevation (DTM or DSM) for the point.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/outdoors', center: [12.694, 47.074], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const popup = new maplibregl.Popup({ closeButton: false });

map.on('click', async (e) => {
  const modell = document.getElementById('modell').value; // dtm or dsm
  const url = `https://api.mapx.at/v1/elevation?points=${e.lngLat.lng.toFixed(6)},${e.lngLat.lat.toFixed(6)}&model=${modell}`;
  try {
    const { features } = await (await fetch(url, AUTH)).json();
    const h = features[0].properties.elevation;
    popup.setLngLat(e.lngLat).setText(h == null ? (de ? 'Außerhalb der Daten' : 'Outside the data') : `${h.toLocaleString(de ? 'de-AT' : 'en')} m (${modell.toUpperCase()})`).addTo(map);
  } catch (err) {
    popup.setLngLat(e.lngLat).setText((de ? 'Fehler: ' : 'Error: ') + err.message).addTo(map);
  }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Query elevation: a click asks /v1/elevation (DTM or DSM) for the point.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a> (ALS-DTM / ALS-DOM 1 m), <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/outdoors/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const label = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
label.style.cssText = 'background:#fff;color:#000;padding:4px 8px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006';
const popup = new ol.Overlay({ element: label, positioning: 'bottom-center', offset: [0, -8] });
const map = new ol.Map({ target: 'map', layers: [base], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([12.694, 47.074]), zoom: 13 }) }); // Großglockner

map.on('click', async (e) => {
  const [lng, lat] = ol.proj.toLonLat(e.coordinate);
  const modell = document.getElementById('modell').value; // dtm or dsm
  const url = `https://api.mapx.at/v1/elevation?points=${lng.toFixed(6)},${lat.toFixed(6)}&model=${modell}`;
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const h = (await r.json()).features[0].properties.elevation;
    label.textContent = h == null ? (de ? 'Außerhalb der Daten' : 'Outside the data') : `${h.toLocaleString(de ? 'de-AT' : 'en')} m (${modell.toUpperCase()})`;
  } catch (err) { label.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
  popup.setPosition(e.coordinate);
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Query elevation: a click asks /v1/elevation (DTM or DSM) for the point.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the elevation model (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a> (ALS-DTM / ALS-DOM 1 m), <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.074, 12.694], 13); // Großglockner
L.tileLayer('https://api.mapx.at/v1/styles/outdoors/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const popup = L.popup({ closeButton: false });
const show = (latlng, s) => popup.setLatLng(latlng).setContent(Object.assign(document.createElement('span'), { textContent: s })).openOn(map);
map.on('click', async (e) => {
  const modell = document.getElementById('modell').value; // dtm or dsm
  const url = `https://api.mapx.at/v1/elevation?points=${e.latlng.lng.toFixed(6)},${e.latlng.lat.toFixed(6)}&model=${modell}`;
  try {
    const res = await fetch(url, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const h = (await res.json()).features[0].properties.elevation;
    show(e.latlng, h == null ? (de ? 'Außerhalb der Daten' : 'Outside the data') : `${h.toLocaleString(de ? 'de-AT' : 'en')} m (${modell.toUpperCase()})`);
  } catch (err) { show(e.latlng, (de ? 'Fehler: ' : 'Error: ') + err.message); }
});
````

## APIs used

- [Elevation](https://docs.mapx.at/api/elevation.md) `GET /v1/elevation`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

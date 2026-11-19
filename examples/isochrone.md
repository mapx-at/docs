# Isochrone example: reachability map

Source: https://docs.mapx.at/examples/isochrone/

> Which area can be reached in 5, 10 and 15 minutes? Draw isochrones (reachability) with the mapx routing API; runnable code to copy.

A click sets the start point. The isochrone API returns one MultiPolygon per step, the largest area first.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Isochrones: areas reachable in 5, 10 and 15 minutes.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.33, 48.23], zoom: 13 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

map.on('load', () => {
  map.addSource('iso', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'iso-flaeche', type: 'fill', source: 'iso', paint: { 'fill-color': '#2346d8', 'fill-opacity': 0.12 } });
  map.addLayer({ id: 'iso-linie', type: 'line', source: 'iso', paint: { 'line-color': '#2346d8', 'line-width': 2 } });
  abfragen([15.33, 48.23]);
});
map.on('click', (e) => abfragen([e.lngLat.lng, e.lngLat.lat]));

async function abfragen(ort) {
  const profil = document.getElementById('profil').value; // walking, cycling or driving
  const url = `https://api.mapx.at/v1/isochrone/${profil}/${ort.join(',')}?contours_minutes=5,10,15`;
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    map.getSource('iso').setData(await r.json()); // one MultiPolygon per step, largest first
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Isochrones: areas reachable in 5, 10 and 15 minutes.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const iso = new ol.source.Vector();
const area = new ol.style.Style({ fill: new ol.style.Fill({ color: 'rgba(35, 70, 216, 0.12)' }), stroke: new ol.style.Stroke({ color: '#2346d8', width: 2 }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: iso, style: area })], view: new ol.View({ center: ol.proj.fromLonLat([15.33, 48.23]), zoom: 14 }) }); // Melk
const geojson = new ol.format.GeoJSON({ featureProjection: 'EPSG:3857' }); // reads lon/lat GeoJSON into map coordinates

map.on('click', (e) => abfragen(ol.proj.toLonLat(e.coordinate)));
abfragen([15.33, 48.23]);

async function abfragen(ort) {
  const profil = document.getElementById('profil').value; // walking, cycling or driving
  const url = `https://api.mapx.at/v1/isochrone/${profil}/${ort.join(',')}?contours_minutes=5,10,15`;
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    iso.clear();
    iso.addFeatures(geojson.readFeatures(await r.json())); // one MultiPolygon per step, largest first
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Isochrones: areas reachable in 5, 10 and 15 minutes.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the routing (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([48.23, 15.33], 14); // Melk
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const iso = L.geoJSON(null, { style: { color: '#2346d8', weight: 2, opacity: 1, fillColor: '#2346d8', fillOpacity: 0.12 } }).addTo(map);
map.on('click', (e) => abfragen([e.latlng.lng, e.latlng.lat]));
abfragen([15.33, 48.23]);
async function abfragen(ort) {
  const profil = document.getElementById('profil').value; // walking, cycling or driving
  const url = `https://api.mapx.at/v1/isochrone/${profil}/${ort.join(',')}?contours_minutes=5,10,15`;
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    iso.clearLayers().addData(await r.json()); // one MultiPolygon per step, largest first
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

## APIs used

- [Routing](https://docs.mapx.at/api/directions.md#isochrones) `GET api.mapx.at/v1/isochrone/{profile}/{coordinate}`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

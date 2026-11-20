# Route: Beispiel mit Start und Ziel

Source: https://docs.mapx.at/de/examples/route/

> Start und Ziel auf der Karte anklicken und die Route samt Länge und Dauer von der mapx-Routing-API erhalten; lauffähiger Code zum Kopieren.

Die Routing-API arbeitet auf dem GIP.at-Netz für Fuß, Rad, Mountainbike und Auto. Die Antwort hat die Mapbox-Form `routes[0]` mit `geometry` (GeoJSON-Linie), `distance` in Metern und `duration` in Sekunden.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Route: click start and destination on the map, the Directions API returns a GeoJSON line.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.33, 48.23], zoom: 13 }); // Melk
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
let a = null, b = null, markerA, markerB;

map.on('load', () => {
  map.addSource('route', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'route', type: 'line', source: 'route', layout: { 'line-cap': 'round', 'line-join': 'round' }, paint: { 'line-color': '#2346d8', 'line-width': 5 } });
});
map.on('click', async (e) => {
  if (a && b) { a = b = null; markerA?.remove(); markerB?.remove(); map.getSource('route').setData({ type: 'FeatureCollection', features: [] }); }
  const p = [e.lngLat.lng, e.lngLat.lat];
  if (!a) { a = p; markerA = new maplibregl.Marker({ color: '#1d7f4e' }).setLngLat(p).addTo(map); return; }
  b = p; markerB = new maplibregl.Marker({ color: '#c8372d' }).setLngLat(p).addTo(map);
  const profil = document.getElementById('profil').value; // walking, cycling, cycling-mtb or driving
  const url = `https://api.mapx.at/v1/directions/${profil}/${a.join(',')};${b.join(',')}?geometries=geojson&overview=full`;
  const info = document.getElementById('info');
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('no route (HTTP ' + r.status + ')');
    const route = (await r.json()).routes[0]; // Mapbox shape: routes[0].geometry is a GeoJSON line
    map.getSource('route').setData({ type: 'Feature', geometry: route.geometry, properties: {} });
    info.textContent = `${(route.distance / 1000).toFixed(2)} km, ${Math.round(route.duration / 60)} min`;
  } catch (err) { info.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Route: click start and destination on the map, the Directions API returns a GeoJSON line.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const route = new ol.source.Vector(); // start, destination and the route line
const line = new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#2346d8', width: 5, lineCap: 'round', lineJoin: 'round' }) });
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: route, style: line })], view: new ol.View({ center: ol.proj.fromLonLat([15.33, 48.23]), zoom: 14 }) }); // Melk
let a = null, b = null;

map.on('click', async (e) => {
  if (a && b) { a = b = null; route.clear(); }
  const p = ol.proj.toLonLat(e.coordinate);
  const marker = new ol.Feature(new ol.geom.Point(e.coordinate));
  route.addFeature(marker);
  if (!a) { a = p; marker.setStyle(pin('#1d7f4e')); return; }
  b = p; marker.setStyle(pin('#c8372d'));
  const profil = document.getElementById('profil').value; // walking, cycling, cycling-mtb or driving
  const url = `https://api.mapx.at/v1/directions/${profil}/${a.join(',')};${b.join(',')}?geometries=geojson&overview=full`;
  const info = document.getElementById('info');
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('no route (HTTP ' + r.status + ')');
    const res = (await r.json()).routes[0]; // Mapbox shape: routes[0].geometry is a GeoJSON line
    route.addFeature(new ol.format.GeoJSON().readFeature(res.geometry, { featureProjection: 'EPSG:3857' }));
    info.textContent = `${(res.distance / 1000).toFixed(2)} km, ${Math.round(res.duration / 60)} min`;
  } catch (err) { info.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Route: click start and destination on the map, the Directions API returns a GeoJSON line.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([48.23, 15.33], 14); // Melk
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const line = L.geoJSON(null, { style: { color: '#2346d8', weight: 5, opacity: 1 } }).addTo(map);
const markers = L.layerGroup().addTo(map);
const pin = (latlng, color) => L.circleMarker(latlng, { radius: 8, color: '#fff', weight: 2, fillColor: color, fillOpacity: 1 }).addTo(markers);
let a = null, b = null;

map.on('click', async (e) => {
  if (a && b) { a = b = null; markers.clearLayers(); line.clearLayers(); }
  const p = [e.latlng.lng, e.latlng.lat];
  if (!a) { a = p; pin(e.latlng, '#1d7f4e'); return; }
  b = p; pin(e.latlng, '#c8372d');
  const profil = document.getElementById('profil').value; // walking, cycling, cycling-mtb or driving
  const url = `https://api.mapx.at/v1/directions/${profil}/${a.join(',')};${b.join(',')}?geometries=geojson&overview=full`;
  const info = document.getElementById('info');
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('no route (HTTP ' + r.status + ')');
    const route = (await r.json()).routes[0]; // Mapbox shape: routes[0].geometry is a GeoJSON line
    line.addData(route.geometry);
    info.textContent = `${(route.distance / 1000).toFixed(2)} km, ${Math.round(route.duration / 60)} min`;
  } catch (err) { info.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

## Verwendete APIs

- [Routing](https://docs.mapx.at/de/api/directions.md#routing) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

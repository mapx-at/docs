# Lkw-Route: Beispiel mit Optionen

Source: https://docs.mapx.at/de/examples/truck-route/

> Lkw-Routing mit Fahrzeugmaßen, Maut und Autobahn meiden sowie Alternativen: lauffähiges Beispiel für die mapx-Routing-API zum Kopieren.

Das Profil `driving-truck` mit Gewicht und Höhe wählen: Straßen mit passenden Verboten und Beschränkungen werden gemieden, die Antwort nennt die GO-mautpflichtigen Kilometer. Für Pkw (`driving`) lassen sich Maut und Autobahn meiden, dazu gibt es bis zu zwei Alternativen. Start und Ziel anklicken.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Truck route: start and destination by click, vehicle size, avoid tolls and motorways, alternatives in grey.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.4, 48.2], zoom: 8 });
map.addControl(new maplibregl.NavigationControl());
const $ = (id) => document.getElementById(id);
const pts = [], markers = [];

map.on('load', () => {
  map.addSource('routes', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'routes', type: 'line', source: 'routes', layout: { 'line-cap': 'round' }, paint: { 'line-color': ['case', ['==', ['get', 'main'], 1], '#2346d8', '#9aa4b8'], 'line-width': ['case', ['==', ['get', 'main'], 1], 6, 4] } });
});
map.on('click', (e) => {
  if (pts.length === 2) { pts.length = 0; markers.splice(0).forEach((m) => m.remove()); }
  pts.push([e.lngLat.lng, e.lngLat.lat]);
  markers.push(new maplibregl.Marker({ color: pts.length === 1 ? '#1d7f4e' : '#c8372d' }).setLngLat(e.lngLat).addTo(map));
  if (pts.length === 2) route();
});
for (const id of ['profile', 'toll', 'motorway', 'weight', 'height']) $(id).addEventListener('change', () => pts.length === 2 && route());

async function route() {
  const profile = $('profile').value, q = new URLSearchParams({ overview: 'full', geometries: 'geojson', alternatives: 'true' });
  if (profile === 'driving-truck') { q.set('weight', $('weight').value); q.set('height', $('height').value); q.delete('alternatives'); }
  const exclude = ['toll', 'motorway'].filter((k) => $(k).checked);
  if (profile === 'driving' && exclude.length) q.set('exclude', exclude.join(','));
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/${profile}/${pts.map((p) => p.join(',')).join(';')}?${q}`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const { routes } = await res.json();
    map.getSource('routes').setData({ type: 'FeatureCollection', features: routes.map((r, i) => ({ type: 'Feature', properties: { main: i === 0 ? 1 : 0 }, geometry: r.geometry })).reverse() });
    const r = routes[0];
    $('info').textContent = `${(r.distance / 1000).toFixed(1)} km, ${Math.round(r.duration / 60)} min` + (r.toll?.truck_go_km != null ? `, ${r.toll.truck_go_km.toFixed(1)} km ${de ? 'GO-mautpflichtig' : 'subject to GO toll'}` : '') + (routes.length > 1 ? `, ${routes.length - 1} ${de ? 'Alternative(n)' : 'alternative(s)'}` : '');
  } catch (err) { $('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Truck route: start and destination by click, vehicle size, avoid tolls and motorways, alternatives in grey.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, Maut: ASFINAG, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const routes = new ol.source.Vector(), points = new ol.source.Vector();
const style = (f) => new ol.style.Style({ zIndex: f.get('main'), stroke: new ol.style.Stroke({ color: f.get('main') ? '#2346d8' : '#9aa4b8', width: f.get('main') ? 6 : 4, lineCap: 'round' }) });
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: routes, style }), new ol.layer.Vector({ source: points })], view: new ol.View({ center: ol.proj.fromLonLat([15.4, 48.2]), zoom: 9 }) });
const geojson = new ol.format.GeoJSON({ featureProjection: 'EPSG:3857' }); // reads lon/lat GeoJSON into map coordinates
const $ = (id) => document.getElementById(id);
const pts = [];

map.on('click', (e) => {
  if (pts.length === 2) { pts.length = 0; points.clear(); routes.clear(); }
  pts.push(ol.proj.toLonLat(e.coordinate));
  const m = new ol.Feature(new ol.geom.Point(e.coordinate));
  m.setStyle(pin(pts.length === 1 ? '#1d7f4e' : '#c8372d'));
  points.addFeature(m);
  if (pts.length === 2) route();
});
for (const id of ['profile', 'toll', 'motorway', 'weight', 'height']) $(id).addEventListener('change', () => pts.length === 2 && route());

async function route() {
  const profile = $('profile').value, q = new URLSearchParams({ overview: 'full', geometries: 'geojson', alternatives: 'true' });
  if (profile === 'driving-truck') { q.set('weight', $('weight').value); q.set('height', $('height').value); q.delete('alternatives'); }
  const exclude = ['toll', 'motorway'].filter((k) => $(k).checked);
  if (profile === 'driving' && exclude.length) q.set('exclude', exclude.join(','));
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/${profile}/${pts.map((p) => p.join(',')).join(';')}?${q}`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const { routes: list } = await res.json();
    routes.clear();
    routes.addFeatures(list.map((r, i) => { const f = geojson.readFeature(r.geometry); f.set('main', i === 0 ? 1 : 0); return f; }));
    const r = list[0];
    $('info').textContent = `${(r.distance / 1000).toFixed(1)} km, ${Math.round(r.duration / 60)} min` + (r.toll?.truck_go_km != null ? `, ${r.toll.truck_go_km.toFixed(1)} km ${de ? 'GO-mautpflichtig' : 'subject to GO toll'}` : '') + (list.length > 1 ? `, ${list.length - 1} ${de ? 'Alternative(n)' : 'alternative(s)'}` : '');
  } catch (err) { $('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Truck route: start and destination by click, vehicle size, avoid tolls and motorways, alternatives in grey.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style, the routing and the toll data (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, Maut: ASFINAG, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([48.2, 15.4], 9);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const $ = (id) => document.getElementById(id);
const lines = L.geoJSON(null, { style: (f) => (f.properties.main ? { color: '#2346d8', weight: 6, opacity: 1 } : { color: '#9aa4b8', weight: 4, opacity: 1 }) }).addTo(map);
const markers = L.layerGroup().addTo(map);
const pts = [];
map.on('click', (e) => {
  if (pts.length === 2) { pts.length = 0; markers.clearLayers(); lines.clearLayers(); }
  pts.push([e.latlng.lng, e.latlng.lat]);
  L.circleMarker(e.latlng, { radius: 8, color: '#fff', weight: 2, fillColor: pts.length === 1 ? '#1d7f4e' : '#c8372d', fillOpacity: 1 }).addTo(markers);
  if (pts.length === 2) route();
});
for (const id of ['profile', 'toll', 'motorway', 'weight', 'height']) $(id).addEventListener('change', () => pts.length === 2 && route());
async function route() {
  const profile = $('profile').value, q = new URLSearchParams({ overview: 'full', geometries: 'geojson', alternatives: 'true' });
  if (profile === 'driving-truck') { q.set('weight', $('weight').value); q.set('height', $('height').value); q.delete('alternatives'); }
  const exclude = ['toll', 'motorway'].filter((k) => $(k).checked);
  if (profile === 'driving' && exclude.length) q.set('exclude', exclude.join(','));
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/${profile}/${pts.map((p) => p.join(',')).join(';')}?${q}`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const { routes } = await res.json();
    lines.clearLayers();
    // alternatives first, the main route last so it is drawn on top
    lines.addData(routes.map((r, i) => ({ type: 'Feature', properties: { main: i === 0 }, geometry: r.geometry })).reverse());
    const r = routes[0];
    $('info').textContent = `${(r.distance / 1000).toFixed(1)} km, ${Math.round(r.duration / 60)} min` + (r.toll?.truck_go_km != null ? `, ${r.toll.truck_go_km.toFixed(1)} km ${de ? 'GO-mautpflichtig' : 'subject to GO toll'}` : '') + (routes.length > 1 ? `, ${routes.length - 1} ${de ? 'Alternative(n)' : 'alternative(s)'}` : '');
  } catch (err) { $('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

## Verwendete APIs

- [Directions](https://docs.mapx.at/de/api/directions.md) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

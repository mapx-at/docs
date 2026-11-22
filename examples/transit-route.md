# Public transit route example

Source: https://docs.mapx.at/examples/transit-route/

> Click start and destination and get the walking and riding legs of a public transport journey in Austria with the mapx transit API.

The transit API combines walks on the road network with rides from the timetable. Without `depart_at` it plans for now; `arrive_by` plans backwards. Walking legs are drawn dashed, rides solid.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Public transit: click start and destination, get walking and riding legs for the next departure.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const pts = [], markers = [];

map.on('load', () => {
  map.addSource('legs', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  const line = { 'line-width': 5 };
  map.addLayer({ id: 'legs-ride', type: 'line', source: 'legs', filter: ['!=', ['get', 'mode'], 'walking'], layout: { 'line-cap': 'round', 'line-join': 'round' }, paint: { ...line, 'line-color': '#2346d8' } });
  map.addLayer({ id: 'legs-walk', type: 'line', source: 'legs', filter: ['==', ['get', 'mode'], 'walking'], layout: { 'line-join': 'round' }, paint: { ...line, 'line-color': '#667', 'line-dasharray': [1, 1.5] } });
});
map.on('click', async (e) => {
  if (pts.length === 2) { pts.length = 0; markers.splice(0).forEach((m) => m.remove()); }
  pts.push([e.lngLat.lng, e.lngLat.lat]);
  markers.push(new maplibregl.Marker({ color: pts.length === 1 ? '#1d7f4e' : '#c8372d' }).setLngLat(e.lngLat).addTo(map));
  if (pts.length < 2) return;
  const info = document.getElementById('info');
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/transit/${pts.map((p) => p.join(',')).join(';')}?routes=1`, AUTH); // add depart_at=2026-11-09T08:00 for a fixed time
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const route = (await res.json()).routes[0];
    map.getSource('legs').setData({ type: 'FeatureCollection', features: route.legs.map((l) => ({ type: 'Feature', properties: { mode: l.mode }, geometry: l.geometry })) });
    info.textContent = `${route.departure.slice(11, 16)} ${de ? 'bis' : 'to'} ${route.arrival.slice(11, 16)}, ${Math.round(route.duration / 60)} min, ${route.transfers} ${de ? 'Umstieg(e)' : 'transfer(s)'}: ` + route.legs.map((l) => l.summary).join(' > ');
  } catch (err) { info.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Public transit: click start and destination, get walking and riding legs for the next departure.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>; Fahrplan/timetable: ÖBB-Personenverkehr AG, Mobilitätsverbünde Österreich OG; <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const legs = new ol.source.Vector(), points = new ol.source.Vector();
const ride = new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#2346d8', width: 5, lineCap: 'round', lineJoin: 'round' }) });
const walk = new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#667', width: 5, lineJoin: 'round', lineDash: [5, 7.5] }) });
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: legs, style: (f) => (f.get('mode') === 'walking' ? walk : ride) }), new ol.layer.Vector({ source: points })], view: new ol.View({ center: ol.proj.fromLonLat([16.3725, 48.2085]), zoom: 13 }) });
const geojson = new ol.format.GeoJSON({ featureProjection: 'EPSG:3857' }); // reads lon/lat GeoJSON into map coordinates
const pts = [];

map.on('click', async (e) => {
  if (pts.length === 2) { pts.length = 0; points.clear(); legs.clear(); }
  pts.push(ol.proj.toLonLat(e.coordinate));
  const m = new ol.Feature(new ol.geom.Point(e.coordinate));
  m.setStyle(pin(pts.length === 1 ? '#1d7f4e' : '#c8372d'));
  points.addFeature(m);
  if (pts.length < 2) return;
  const info = document.getElementById('info');
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/transit/${pts.map((p) => p.join(',')).join(';')}?routes=1`, AUTH); // add depart_at=2026-11-09T08:00 for a fixed time
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const route = (await res.json()).routes[0];
    legs.addFeatures(route.legs.map((l) => { const f = geojson.readFeature(l.geometry); f.set('mode', l.mode); return f; }));
    info.textContent = `${route.departure.slice(11, 16)} ${de ? 'bis' : 'to'} ${route.arrival.slice(11, 16)}, ${Math.round(route.duration / 60)} min, ${route.transfers} ${de ? 'Umstieg(e)' : 'transfer(s)'}: ` + route.legs.map((l) => l.summary).join(' > ');
  } catch (err) { info.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Public transit: click start and destination, get walking and riding legs for the next departure.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the timetables, must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>; Fahrplan: ÖBB-Personenverkehr AG (CC BY 4.0), Mobilitätsverbünde Österreich OG';

const map = L.map('map').setView([48.2085, 16.3725], 13);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
// riding legs solid blue, walking legs dashed grey
const legs = L.geoJSON(null, { style: (f) => (f.properties.mode === 'walking' ? { color: '#667', weight: 5, opacity: 1, dashArray: '5 8' } : { color: '#2346d8', weight: 5, opacity: 1 }) }).addTo(map);
const markers = L.layerGroup().addTo(map);
const pts = [];
map.on('click', async (e) => {
  if (pts.length === 2) { pts.length = 0; markers.clearLayers(); legs.clearLayers(); }
  pts.push([e.latlng.lng, e.latlng.lat]);
  L.circleMarker(e.latlng, { radius: 8, color: '#fff', weight: 2, fillColor: pts.length === 1 ? '#1d7f4e' : '#c8372d', fillOpacity: 1 }).addTo(markers);
  if (pts.length < 2) return;
  const info = document.getElementById('info');
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/transit/${pts.map((p) => p.join(',')).join(';')}?routes=1`, AUTH); // add depart_at=2026-11-09T08:00 for a fixed time
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const route = (await res.json()).routes[0];
    legs.addData(route.legs.map((l) => ({ type: 'Feature', properties: { mode: l.mode }, geometry: l.geometry })));
    info.textContent = `${route.departure.slice(11, 16)} ${de ? 'bis' : 'to'} ${route.arrival.slice(11, 16)}, ${Math.round(route.duration / 60)} min, ${route.transfers} ${de ? 'Umstieg(e)' : 'transfer(s)'}: ` + route.legs.map((l) => l.summary).join(' > ');
  } catch (err) { info.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

## APIs used

- [Transit](https://docs.mapx.at/api/transit.md) `GET api.mapx.at/v1/directions/transit/{coordinates}`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

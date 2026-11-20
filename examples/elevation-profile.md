# Elevation profile example on a map

Source: https://docs.mapx.at/examples/elevation-profile/

> Click a line on the map and see its elevation profile (Höhenprofil): elevation profile API with runnable MapLibre GL JS code to copy.

The vertices go to `/v1/elevation/profile` as `coordinates`; it returns `samples` evenly spaced points plus ascent and descent. A canvas draws the profile.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Elevation profile: click a line on the map, /v1/elevation/profile returns the heights, a canvas draws the profile.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/outdoors', center: [12.70, 47.08], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
let punkte = [[12.67, 47.065], [12.71, 47.09]];

map.on('load', () => {
  map.addSource('linie', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'linie', type: 'line', source: 'linie', paint: { 'line-color': '#2346d8', 'line-width': 4 } });
  zeichne();
});
map.on('click', (e) => { punkte.push([e.lngLat.lng, e.lngLat.lat]); zeichne(); });
document.getElementById('neu').addEventListener('click', () => { punkte = []; zeichne(); });

async function zeichne() {
  map.getSource('linie').setData({ type: 'Feature', geometry: { type: 'LineString', coordinates: punkte } });
  const canvas = document.getElementById('profil'), g = canvas.getContext('2d');
  g.clearRect(0, 0, canvas.width, canvas.height);
  if (punkte.length < 2) { document.getElementById('info').textContent = de ? 'Mindestens zwei Punkte setzen.' : 'Set at least two points.'; return; }
  const pfad = punkte.map((p) => p.map((v) => v.toFixed(5)).join(',')).join(';');
  const antwort = await fetch(`https://api.mapx.at/v1/elevation/profile?coordinates=${pfad}&samples=120`, AUTH);
  const { distances, elevations } = (await antwort.json()).properties;
  const pts = distances.map((d, i) => ({ d, h: elevations[i] })).filter((p) => p.h != null);
  const dMax = pts.at(-1).d, hMin = Math.min(...pts.map((p) => p.h)), hMax = Math.max(...pts.map((p) => p.h));
  const W = canvas.width, H = canvas.height, x = (d) => 50 + (d / dMax) * (W - 60), y = (h) => H - 24 - ((h - hMin) / (hMax - hMin || 1)) * (H - 40);
  g.beginPath(); pts.forEach((p, i) => (i ? g.lineTo(x(p.d), y(p.h)) : g.moveTo(x(p.d), y(p.h))));
  g.strokeStyle = '#2346d8'; g.lineWidth = 2; g.stroke();
  g.fillStyle = '#6b7079'; g.font = '12px sans-serif';
  g.fillText(`${Math.round(hMax)} m`, 4, 16); g.fillText(`${Math.round(hMin)} m`, 4, H - 28); g.fillText(`${(dMax / 1000).toFixed(2)} km`, W - 60, H - 6);
  let auf = 0, ab = 0; pts.forEach((p, i) => { if (i) { const dh = p.h - pts[i - 1].h; dh > 0 ? (auf += dh) : (ab -= dh); } });
  document.getElementById('info').textContent = `${(dMax / 1000).toFixed(2)} km, ${de ? 'Aufstieg' : 'ascent'} ${Math.round(auf)} m, ${de ? 'Abstieg' : 'descent'} ${Math.round(ab)} m`;
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Elevation profile: click a line on the map, /v1/elevation/profile returns the heights, a canvas draws the profile.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a> (ALS-DTM / ALS-DOM 1 m), <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/outdoors/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const linie = new ol.Feature(new ol.geom.LineString([]));
const stroke = new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#2346d8', width: 4 }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: new ol.source.Vector({ features: [linie] }), style: stroke })], view: new ol.View({ center: ol.proj.fromLonLat([12.70, 47.08]), zoom: 13 }) });
let punkte = [[12.67, 47.065], [12.71, 47.09]];

map.on('click', (e) => { punkte.push(ol.proj.toLonLat(e.coordinate)); zeichne(); });
document.getElementById('neu').addEventListener('click', () => { punkte = []; zeichne(); });
zeichne();

async function zeichne() {
  linie.getGeometry().setCoordinates(punkte.map((p) => ol.proj.fromLonLat(p)));
  const canvas = document.getElementById('profil'), g = canvas.getContext('2d');
  g.clearRect(0, 0, canvas.width, canvas.height);
  if (punkte.length < 2) { document.getElementById('info').textContent = de ? 'Mindestens zwei Punkte setzen.' : 'Set at least two points.'; return; }
  const pfad = punkte.map((p) => p.map((v) => v.toFixed(5)).join(',')).join(';');
  try {
    const antwort = await fetch(`https://api.mapx.at/v1/elevation/profile?coordinates=${pfad}&samples=120`, AUTH);
    if (!antwort.ok) throw new Error('HTTP ' + antwort.status);
    const { distances, elevations } = (await antwort.json()).properties;
    const pts = distances.map((d, i) => ({ d, h: elevations[i] })).filter((p) => p.h != null);
    const dMax = pts.at(-1).d, hMin = Math.min(...pts.map((p) => p.h)), hMax = Math.max(...pts.map((p) => p.h));
    const W = canvas.width, H = canvas.height, x = (d) => 50 + (d / dMax) * (W - 60), y = (h) => H - 24 - ((h - hMin) / (hMax - hMin || 1)) * (H - 40);
    g.beginPath(); pts.forEach((p, i) => (i ? g.lineTo(x(p.d), y(p.h)) : g.moveTo(x(p.d), y(p.h))));
    g.strokeStyle = '#2346d8'; g.lineWidth = 2; g.stroke();
    g.fillStyle = '#6b7079'; g.font = '12px sans-serif';
    g.fillText(`${Math.round(hMax)} m`, 4, 16); g.fillText(`${Math.round(hMin)} m`, 4, H - 28); g.fillText(`${(dMax / 1000).toFixed(2)} km`, W - 60, H - 6);
    let auf = 0, ab = 0; pts.forEach((p, i) => { if (i) { const dh = p.h - pts[i - 1].h; dh > 0 ? (auf += dh) : (ab -= dh); } });
    document.getElementById('info').textContent = `${(dMax / 1000).toFixed(2)} km, ${de ? 'Aufstieg' : 'ascent'} ${Math.round(auf)} m, ${de ? 'Abstieg' : 'descent'} ${Math.round(ab)} m`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Elevation profile: click a line on the map, /v1/elevation/profile returns the heights, a canvas draws the profile.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the elevation model (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a> (ALS-DTM 1 m), <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.08, 12.70], 13);
L.tileLayer('https://api.mapx.at/v1/styles/outdoors/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
let punkte = [[12.67, 47.065], [12.71, 47.09]]; // lng, lat
const linie = L.polyline([], { color: '#2346d8', weight: 4, opacity: 1 }).addTo(map);
map.on('click', (e) => { punkte.push([e.latlng.lng, e.latlng.lat]); zeichne(); });
document.getElementById('neu').addEventListener('click', () => { punkte = []; zeichne(); });
zeichne();
async function zeichne() {
  linie.setLatLngs(punkte.map(([lng, lat]) => [lat, lng]));
  const canvas = document.getElementById('profil'), g = canvas.getContext('2d');
  g.clearRect(0, 0, canvas.width, canvas.height);
  if (punkte.length < 2) { document.getElementById('info').textContent = de ? 'Mindestens zwei Punkte setzen.' : 'Set at least two points.'; return; }
  const pfad = punkte.map((p) => p.map((v) => v.toFixed(5)).join(',')).join(';');
  try {
    const antwort = await fetch(`https://api.mapx.at/v1/elevation/profile?coordinates=${pfad}&samples=120`, AUTH);
    if (!antwort.ok) throw new Error('HTTP ' + antwort.status);
    const { distances, elevations } = (await antwort.json()).properties;
    const pts = distances.map((d, i) => ({ d, h: elevations[i] })).filter((p) => p.h != null);
    const dMax = pts.at(-1).d, hMin = Math.min(...pts.map((p) => p.h)), hMax = Math.max(...pts.map((p) => p.h));
    const W = canvas.width, H = canvas.height, x = (d) => 50 + (d / dMax) * (W - 60), y = (h) => H - 24 - ((h - hMin) / (hMax - hMin || 1)) * (H - 40);
    g.beginPath(); pts.forEach((p, i) => (i ? g.lineTo(x(p.d), y(p.h)) : g.moveTo(x(p.d), y(p.h))));
    g.strokeStyle = '#2346d8'; g.lineWidth = 2; g.stroke();
    g.fillStyle = '#6b7079'; g.font = '12px sans-serif';
    g.fillText(`${Math.round(hMax)} m`, 4, 16); g.fillText(`${Math.round(hMin)} m`, 4, H - 28); g.fillText(`${(dMax / 1000).toFixed(2)} km`, W - 60, H - 6);
    let auf = 0, ab = 0; pts.forEach((p, i) => { if (i) { const dh = p.h - pts[i - 1].h; dh > 0 ? (auf += dh) : (ab -= dh); } });
    document.getElementById('info').textContent = `${(dMax / 1000).toFixed(2)} km, ${de ? 'Aufstieg' : 'ascent'} ${Math.round(auf)} m, ${de ? 'Abstieg' : 'descent'} ${Math.round(ab)} m`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

## APIs used

- [Elevation and profile](https://docs.mapx.at/api/elevation.md#elevation-profile-along-a-line) `GET api.mapx.at/v1/elevation/profile`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

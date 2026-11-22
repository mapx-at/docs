# Fahrzeitmatrix: Beispiel

Source: https://docs.mapx.at/de/examples/matrix/

> Fahrzeiten zwischen mehreren Orten in einer Anfrage mit der mapx-Matrix-API berechnen (Fahrzeitmatrix): lauffähiges Beispiel zum Kopieren.

Die Matrix-API nimmt Quellen (und optional Ziele) und liefert Dauer (und optional Distanz) für alle Paare, `null` heißt nicht routbar.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Matrix: Fahrzeiten zwischen mehreren Orten in einer Anfrage.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const ORTE = { Melk: [15.33, 48.228], 'St. Pölten': [15.626, 48.204], Krems: [15.6, 48.41], Amstetten: [14.87, 48.123] };
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.3, 48.25], zoom: 8 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
for (const [name, p] of Object.entries(ORTE)) new maplibregl.Marker().setLngLat(p).setPopup(new maplibregl.Popup().setText(name)).addTo(map);

document.getElementById('los').addEventListener('click', async () => {
  const tabelle = document.getElementById('ergebnis'), namen = Object.keys(ORTE);
  try {
    const koordinaten = Object.values(ORTE).map((p) => p.join(',')).join(';');
    const r = await fetch(`https://api.mapx.at/v1/matrix/driving/${koordinaten}`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const { durations } = await r.json(); // seconds, null = not routable
    tabelle.innerHTML = '<tr><th></th>' + namen.map((n) => `<th>${n}</th>`).join('') + '</tr>' +
      durations.map((zeile, i) => `<tr><th>${namen[i]}</th>` + zeile.map((s) => `<td>${s == null ? '–' : Math.round(s / 60) + ' min'}</td>`).join('') + '</tr>').join('');
  } catch (err) { tabelle.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Matrix: driving times between several places in one request.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const ORTE = { Melk: [15.33, 48.228], 'St. Pölten': [15.626, 48.204], Krems: [15.6, 48.41], Amstetten: [14.87, 48.123] };
const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const places = new ol.source.Vector({ features: Object.entries(ORTE).map(([name, p]) => new ol.Feature({ geometry: new ol.geom.Point(ol.proj.fromLonLat(p)), name })) });
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const label = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
label.style.cssText = 'background:#fff;color:#000;padding:4px 8px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006';
const popup = new ol.Overlay({ element: label, positioning: 'bottom-center', offset: [0, -8] });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: places, style: pin('#3fb1ce') })], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([15.3, 48.25]), zoom: 9 }) });
map.on('click', (e) => {
  const f = map.forEachFeatureAtPixel(e.pixel, (x) => x);
  label.textContent = f?.get('name') ?? '';
  popup.setPosition(f?.getGeometry().getCoordinates());
});

document.getElementById('los').addEventListener('click', async () => {
  const tabelle = document.getElementById('ergebnis'), namen = Object.keys(ORTE);
  const zelle = (tag, text) => Object.assign(document.createElement(tag), { textContent: text });
  const zeile = (kopf, zellen, tag = 'td') => tabelle.insertRow().append(zelle('th', kopf), ...zellen.map((z) => zelle(tag, z)));
  try {
    const koordinaten = Object.values(ORTE).map((p) => p.join(',')).join(';');
    const r = await fetch(`https://api.mapx.at/v1/matrix/driving/${koordinaten}`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const { durations } = await r.json(); // seconds, null = not routable
    tabelle.replaceChildren();
    zeile('', namen, 'th');
    durations.forEach((z, i) => zeile(namen[i], z.map((s) => (s == null ? '–' : Math.round(s / 60) + ' min'))));
  } catch (err) { tabelle.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Matrix: driving times between several places in one request.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the routing (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const ORTE = { Melk: [15.33, 48.228], 'St. Pölten': [15.626, 48.204], Krems: [15.6, 48.41], Amstetten: [14.87, 48.123] };
const map = L.map('map').setView([48.25, 15.3], 9);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
for (const [name, [lng, lat]] of Object.entries(ORTE)) L.circleMarker([lat, lng], { radius: 8, color: '#fff', weight: 2, fillColor: '#2346d8', fillOpacity: 1 }).bindPopup(Object.assign(document.createElement('span'), { textContent: name })).addTo(map);
document.getElementById('los').addEventListener('click', async () => {
  const tabelle = document.getElementById('ergebnis'), namen = Object.keys(ORTE);
  const zeile = (kopf, zellen) => { const tr = document.createElement('tr'); tr.append(Object.assign(document.createElement('th'), { textContent: kopf }), ...zellen.map((z) => Object.assign(document.createElement(z.th ? 'th' : 'td'), { textContent: z.text }))); return tr; };
  try {
    const koordinaten = Object.values(ORTE).map((p) => p.join(',')).join(';');
    const r = await fetch(`https://api.mapx.at/v1/matrix/driving/${koordinaten}`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const { durations } = await r.json(); // seconds, null = not routable
    tabelle.replaceChildren(zeile('', namen.map((n) => ({ th: true, text: n }))), ...durations.map((z, i) => zeile(namen[i], z.map((s) => ({ text: s == null ? '–' : Math.round(s / 60) + ' min' })))));
  } catch (err) { tabelle.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

## Verwendete APIs

- [Matrix](https://docs.mapx.at/de/api/matrix.md) `GET api.mapx.at/v1/matrix/{profile}/{coordinates}`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

# Search Box example with suggestions

Source: https://docs.mapx.at/examples/search-box/

> Suggestions while typing, position after the click and a type filter: type-ahead search with the mapx Search Box API, runnable code.

Search Box works in two steps: `suggest` returns suggestions while the user types, `retrieve` returns the position of the chosen one. Use the same `session_token` for both calls and a new one for the next search. The type filter limits the suggestions.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Search Box: suggestions while typing (suggest), position and details after the click (retrieve). One session token per search.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [14.3, 47.6], zoom: 7 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const input = document.getElementById('q'), types = document.getElementById('types'), list = document.getElementById('treffer');
let marker, token = crypto.randomUUID();

input.addEventListener('input', async () => {
  list.replaceChildren();
  if (input.value.trim().length < 2) return;
  const c = map.getCenter();
  const url = `https://api.mapx.at/v1/searchbox/suggest?q=${encodeURIComponent(input.value)}&session_token=${token}&proximity=${c.lng},${c.lat}&limit=6` + (types.value ? `&types=${types.value}` : '');
  try {
    const res = await fetch(url, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    for (const s of (await res.json()).suggestions) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = `${s.name} (${s.feature_type}) ${s.place_formatted ?? ''}`;
      b.addEventListener('click', () => pick(s.mapbox_id));
      li.append(b); list.append(li);
    }
  } catch (e) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});

async function pick(id) {
  const res = await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${encodeURIComponent(id)}?session_token=${token}`, AUTH);
  const f = (await res.json()).features[0];
  marker?.remove();
  marker = new maplibregl.Marker().setLngLat(f.geometry.coordinates).addTo(map);
  map.flyTo({ center: f.geometry.coordinates, zoom: f.properties.zoom ?? 15 });
  list.replaceChildren();
  token = crypto.randomUUID(); // the session ends with retrieve
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Search Box: suggestions while typing (suggest), position and details after the click (retrieve). One session token per search.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const hit = new ol.source.Vector(); // the chosen result
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: hit, style: pin('#3fb1ce') })], view: new ol.View({ center: ol.proj.fromLonLat([14.3, 47.6]), zoom: 8 }) });
const input = document.getElementById('q'), types = document.getElementById('types'), list = document.getElementById('treffer');
let token = crypto.randomUUID();

input.addEventListener('input', async () => {
  list.replaceChildren();
  if (input.value.trim().length < 2) return;
  const [lon, lat] = ol.proj.toLonLat(map.getView().getCenter());
  const url = `https://api.mapx.at/v1/searchbox/suggest?q=${encodeURIComponent(input.value)}&session_token=${token}&proximity=${lon},${lat}&limit=6` + (types.value ? `&types=${types.value}` : '');
  try {
    const res = await fetch(url, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    for (const s of (await res.json()).suggestions) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = `${s.name} (${s.feature_type}) ${s.place_formatted ?? ''}`;
      b.addEventListener('click', () => pick(s.mapbox_id));
      li.append(b); list.append(li);
    }
  } catch (e) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});

async function pick(id) {
  try {
    const res = await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${encodeURIComponent(id)}?session_token=${token}`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const f = (await res.json()).features[0];
    const at = ol.proj.fromLonLat(f.geometry.coordinates);
    hit.clear();
    hit.addFeature(new ol.Feature(new ol.geom.Point(at)));
    map.getView().animate({ center: at, zoom: (f.properties.zoom ?? 15) + 1 });
    list.replaceChildren();
  } catch (e) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
  token = crypto.randomUUID(); // the session ends with retrieve
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Search Box: suggestions while typing (suggest), position and details after the click (retrieve). One session token per search.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the search (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.6, 14.3], 8);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const input = document.getElementById('q'), types = document.getElementById('types'), list = document.getElementById('treffer');
let marker, token = crypto.randomUUID();
input.addEventListener('input', async () => {
  list.replaceChildren();
  if (input.value.trim().length < 2) return;
  const c = map.getCenter();
  const url = `https://api.mapx.at/v1/searchbox/suggest?q=${encodeURIComponent(input.value)}&session_token=${token}&proximity=${c.lng},${c.lat}&limit=6` + (types.value ? `&types=${types.value}` : '');
  try {
    const res = await fetch(url, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    for (const s of (await res.json()).suggestions) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = `${s.name} (${s.feature_type}) ${s.place_formatted ?? ''}`;
      b.addEventListener('click', () => pick(s.mapbox_id));
      li.append(b); list.append(li);
    }
  } catch (e) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
async function pick(id) {
  try {
    const res = await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${encodeURIComponent(id)}?session_token=${token}`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const f = (await res.json()).features[0];
    const [lng, lat] = f.geometry.coordinates;
    marker?.remove();
    marker = L.circleMarker([lat, lng], { radius: 8, color: '#fff', weight: 2, fillColor: '#2346d8', fillOpacity: 1 }).addTo(map);
    map.flyTo([lat, lng], (f.properties.zoom ?? 15) + 1);
    list.replaceChildren();
  } catch (e) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
  token = crypto.randomUUID(); // the session ends with retrieve
}
````

## APIs used

- [Search Box](https://docs.mapx.at/api/search-box.md) `GET api.mapx.at/v1/searchbox/suggest`
- [Search Box](https://docs.mapx.at/api/search-box.md) `GET api.mapx.at/v1/searchbox/retrieve/{id}`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

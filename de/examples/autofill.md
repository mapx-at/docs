# Adress-Autofill im Formular: Beispiel

Source: https://docs.mapx.at/de/examples/autofill/

> Straße tippen, Vorschlag wählen, die Formularfelder füllen sich: Adress-Autofill für österreichische Adressen mit der mapx-Autofill-API. Lauffähiger Code.

Autofill ist für Bestell- und Anmeldeformulare gedacht: ein Textfeld, eine Vorschlagsliste, und die übrigen Felder (Hausnummer, PLZ, Ort) werden aus dem gewählten Vorschlag gefüllt. Jeder Vorschlag bringt seine Koordinaten schon mit.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Address autofill: type a street and number, pick a suggestion, the form fields fill themselves.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [13.8, 47.7], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const f = Object.fromEntries(['street', 'housenumber', 'postcode', 'place'].map((k) => [k, document.getElementById(k)]));
const list = document.getElementById('treffer');
let marker;

let ctrl; // AbortController of the running request
f.street.addEventListener('input', async () => {
  ctrl?.abort(); // a newer input cancels the older request
  if (f.street.value.trim().length < 3) { list.replaceChildren(); return; }
  ctrl = new AbortController();
  try {
    const res = await fetch(`https://api.mapx.at/v1/autofill/suggest?q=${encodeURIComponent(f.street.value)}&limit=5&session_token=demo`, { ...AUTH, signal: ctrl.signal });
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const items = (await res.json()).suggestions;
    list.replaceChildren(); // clear only now, so fast typing never shows two lists
    for (const s of items) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = s.full_address;
      b.addEventListener('click', () => {
        for (const k of Object.keys(f)) f[k].value = s[k] ?? '';
        marker?.remove();
        marker = new maplibregl.Marker().setLngLat(s.coordinates).addTo(map);
        map.flyTo({ center: s.coordinates, zoom: 16 });
        list.replaceChildren();
      });
      li.append(b); list.append(li);
    }
  } catch (e) {
    if (e.name === 'AbortError') return; // replaced by a newer input
    list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message;
  }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Address autofill: type a street and number, pick a suggestion, the form fields fill themselves.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const hit = new ol.source.Vector(); // the chosen address
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: hit, style: pin('#3fb1ce') })], view: new ol.View({ center: ol.proj.fromLonLat([13.8, 47.7]), zoom: 7.5 }) });
const f = Object.fromEntries(['street', 'housenumber', 'postcode', 'place'].map((k) => [k, document.getElementById(k)]));
const list = document.getElementById('treffer');

let ctrl; // AbortController of the running request
f.street.addEventListener('input', async () => {
  ctrl?.abort(); // a newer input cancels the older request
  if (f.street.value.trim().length < 3) { list.replaceChildren(); return; }
  ctrl = new AbortController();
  try {
    const res = await fetch(`https://api.mapx.at/v1/autofill/suggest?q=${encodeURIComponent(f.street.value)}&limit=5&session_token=demo`, { ...AUTH, signal: ctrl.signal });
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const items = (await res.json()).suggestions;
    list.replaceChildren(); // clear only now, so fast typing never shows two lists
    for (const s of items) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = s.full_address;
      b.addEventListener('click', () => {
        for (const k of Object.keys(f)) f[k].value = s[k] ?? '';
        const at = ol.proj.fromLonLat(s.coordinates);
        hit.clear();
        hit.addFeature(new ol.Feature(new ol.geom.Point(at)));
        map.getView().setCenter(at), map.getView().setZoom(17); // jump instead of animating: fewer tile requests
        list.replaceChildren();
      });
      li.append(b); list.append(li);
    }
  } catch (e) {
    if (e.name === 'AbortError') return; // replaced by a newer input
    list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message;
  }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Address autofill: type a street and number, pick a suggestion, the form fields fill themselves.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the addresses (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.7, 13.8], 7);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const f = Object.fromEntries(['street', 'housenumber', 'postcode', 'place'].map((k) => [k, document.getElementById(k)]));
const list = document.getElementById('treffer');
let marker;
let ctrl; // AbortController of the running request
f.street.addEventListener('input', async () => {
  ctrl?.abort(); // a newer input cancels the older request
  if (f.street.value.trim().length < 3) { list.replaceChildren(); return; }
  ctrl = new AbortController();
  try {
    const res = await fetch(`https://api.mapx.at/v1/autofill/suggest?q=${encodeURIComponent(f.street.value)}&limit=5&session_token=demo`, { ...AUTH, signal: ctrl.signal });
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const items = (await res.json()).suggestions;
    list.replaceChildren(); // clear only now, so fast typing never shows two lists
    for (const s of items) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = s.full_address;
      b.addEventListener('click', () => {
        for (const k of Object.keys(f)) f[k].value = s[k] ?? '';
        const [lng, lat] = s.coordinates;
        marker?.remove();
        marker = L.circleMarker([lat, lng], { radius: 8, color: '#fff', weight: 2, fillColor: '#2346d8', fillOpacity: 1 }).addTo(map);
        map.setView([lat, lng], 17, { animate: false }); // jump instead of flying: fewer tile requests
        list.replaceChildren();
      });
      li.append(b); list.append(li);
    }
  } catch (e) {
    if (e.name === 'AbortError') return; // replaced by a newer input
    list.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message;
  }
});
````

## Verwendete APIs

- [Adressen](https://docs.mapx.at/de/api/address.md) `GET api.mapx.at/v1/autofill/suggest`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

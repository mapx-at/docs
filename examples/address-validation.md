# Address validation example

Source: https://docs.mapx.at/examples/address-validation/

> Check an address against the official Austrian address register with the address validation API and show the corrected result. Runnable code.

Validation compares the four address parts with the BEV address register. The result has a `status` (`match`, `corrected`, `ambiguous`, `not_found`), the list of `deviations` and either the matching `address` with coordinates or a list of `candidates`. The pre-filled address has a typo on purpose.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Address validation: check a typed address against the official register (BEV) and show the corrected result.
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
const out = document.getElementById('ergebnis');
let marker;

document.getElementById('pruefen').addEventListener('click', async () => {
  const p = new URLSearchParams();
  for (const k of ['street', 'housenumber', 'postcode', 'place']) p.set(k, document.getElementById(k).value);
  try {
    const res = await fetch('https://api.mapx.at/v1/address/validate?' + p, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const r = await res.json(); // status: match, corrected, ambiguous, not_found
    const hit = r.address ? { text: r.address.full_address, at: r.coordinates } : r.candidates?.[0] && { text: r.candidates[0].full_address, at: r.candidates[0].coordinates };
    out.textContent = `Status: ${r.status}` + (r.deviations?.length ? `, ${de ? 'Abweichungen' : 'deviations'}: ${r.deviations.map((d) => `${d.field} ${d.input} > ${d.expected}`).join('; ')}` : '') + (hit ? `, ${r.address ? (de ? 'Adresse' : 'address') : (de ? 'erster Kandidat' : 'first candidate')}: ${hit.text}` : '');
    marker?.remove();
    if (hit) { marker = new maplibregl.Marker().setLngLat(hit.at).addTo(map); map.flyTo({ center: hit.at, zoom: 16 }); }
  } catch (e) { out.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Address validation: check a typed address against the official register (BEV) and show the corrected result.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const hit = new ol.source.Vector(); // the validated address or first candidate
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: hit, style: pin('#3fb1ce') })], view: new ol.View({ center: ol.proj.fromLonLat([13.8, 47.7]), zoom: 7.5 }) });
const out = document.getElementById('ergebnis');

document.getElementById('pruefen').addEventListener('click', async () => {
  const p = new URLSearchParams();
  for (const k of ['street', 'housenumber', 'postcode', 'place']) p.set(k, document.getElementById(k).value);
  try {
    const res = await fetch('https://api.mapx.at/v1/address/validate?' + p, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const r = await res.json(); // status: match, corrected, ambiguous, not_found
    const top = r.address ? { text: r.address.full_address, at: r.coordinates } : r.candidates?.[0] && { text: r.candidates[0].full_address, at: r.candidates[0].coordinates };
    out.textContent = `Status: ${r.status}` + (r.deviations?.length ? `, ${de ? 'Abweichungen' : 'deviations'}: ${r.deviations.map((d) => `${d.field} ${d.input} > ${d.expected}`).join('; ')}` : '') + (top ? `, ${r.address ? (de ? 'Adresse' : 'address') : (de ? 'erster Kandidat' : 'first candidate')}: ${top.text}` : '');
    hit.clear();
    if (top) { hit.addFeature(new ol.Feature(new ol.geom.Point(ol.proj.fromLonLat(top.at)))); map.getView().animate({ center: ol.proj.fromLonLat(top.at), zoom: 17 }); }
  } catch (e) { out.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Address validation: check a typed address against the official register (BEV) and show the corrected result.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the addresses (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.7, 13.8], 7);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const out = document.getElementById('ergebnis');
let marker;
document.getElementById('pruefen').addEventListener('click', async () => {
  const p = new URLSearchParams();
  for (const k of ['street', 'housenumber', 'postcode', 'place']) p.set(k, document.getElementById(k).value);
  try {
    const res = await fetch('https://api.mapx.at/v1/address/validate?' + p, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const r = await res.json(); // status: match, corrected, ambiguous, not_found
    const hit = r.address ? { text: r.address.full_address, at: r.coordinates } : r.candidates?.[0] && { text: r.candidates[0].full_address, at: r.candidates[0].coordinates };
    out.textContent = `Status: ${r.status}` + (r.deviations?.length ? `, ${de ? 'Abweichungen' : 'deviations'}: ${r.deviations.map((d) => `${d.field} ${d.input} > ${d.expected}`).join('; ')}` : '') + (hit ? `, ${r.address ? (de ? 'Adresse' : 'address') : (de ? 'erster Kandidat' : 'first candidate')}: ${hit.text}` : '');
    marker?.remove();
    if (hit) {
      const latlng = [hit.at[1], hit.at[0]];
      marker = L.circleMarker(latlng, { radius: 8, color: '#fff', weight: 2, fillColor: '#2346d8', fillOpacity: 1 }).addTo(map);
      map.flyTo(latlng, 17);
    }
  } catch (e) { out.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

## APIs used

- [Address](https://docs.mapx.at/api/address.md) `GET api.mapx.at/v1/address/validate`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

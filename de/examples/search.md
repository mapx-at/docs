# Ortssuche: Beispiel mit Geocoding

Source: https://docs.mapx.at/de/examples/search/

> Orte, Adressen und Gipfel in Österreich suchen und auf der Karte anspringen: Beispiel zur Geocoding-API mit MapLibre GL JS und lauffähigem Code.

Die Geocoding-API liefert eine GeoJSON-`FeatureCollection` im Mapbox-Stil. Jeder Treffer hat `place_name`, `center` und meist eine `bbox` zum Zoomen.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Search: places, addresses and peaks via /v1/geocode (GeoJSON in the Mapbox shape).
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
let marker;

const eingabe = document.getElementById('q'), liste = document.getElementById('treffer');
eingabe.addEventListener('input', async () => {
  const q = eingabe.value.trim();
  liste.replaceChildren();
  if (q.length < 2) return;
  try {
    const r = await fetch(`https://api.mapx.at/v1/geocode?q=${encodeURIComponent(q)}&limit=6`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    for (const f of (await r.json()).features) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = f.place_name;
      b.addEventListener('click', () => {
        marker?.remove();
        marker = new maplibregl.Marker().setLngLat(f.center).addTo(map);
        f.bbox ? map.fitBounds([[f.bbox[0], f.bbox[1]], [f.bbox[2], f.bbox[3]]], { padding: 60, maxZoom: 17 }) : map.flyTo({ center: f.center, zoom: f.properties.zoom ?? 14 });
        liste.replaceChildren();
      });
      li.append(b); liste.append(li);
    }
  } catch (e) { liste.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Search: places, addresses and peaks via /v1/geocode (GeoJSON in the Mapbox shape).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const hit = new ol.source.Vector(); // the chosen result
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: hit, style: pin('#3fb1ce') })], view: new ol.View({ center: ol.proj.fromLonLat([14.3, 47.6]), zoom: 8 }) });

const eingabe = document.getElementById('q'), liste = document.getElementById('treffer');
eingabe.addEventListener('input', async () => {
  const q = eingabe.value.trim();
  liste.replaceChildren();
  if (q.length < 2) return;
  try {
    const r = await fetch(`https://api.mapx.at/v1/geocode?q=${encodeURIComponent(q)}&limit=6`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    for (const f of (await r.json()).features) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = f.place_name;
      b.addEventListener('click', () => {
        hit.clear();
        hit.addFeature(new ol.Feature(new ol.geom.Point(ol.proj.fromLonLat(f.center))));
        const view = map.getView();
        f.bbox ? view.fit(ol.proj.transformExtent(f.bbox, 'EPSG:4326', 'EPSG:3857'), { padding: [60, 60, 60, 60], maxZoom: 18, duration: 600 }) : view.animate({ center: ol.proj.fromLonLat(f.center), zoom: (f.properties.zoom ?? 14) + 1 });
        liste.replaceChildren();
      });
      li.append(b); liste.append(li);
    }
  } catch (e) { liste.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Search: places, addresses and peaks via /v1/geocode (GeoJSON in the Mapbox shape).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the search (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([47.6, 14.3], 8);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
let marker;
const eingabe = document.getElementById('q'), liste = document.getElementById('treffer');
eingabe.addEventListener('input', async () => {
  const q = eingabe.value.trim();
  liste.replaceChildren();
  if (q.length < 2) return;
  try {
    const r = await fetch(`https://api.mapx.at/v1/geocode?q=${encodeURIComponent(q)}&limit=6`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    for (const f of (await r.json()).features) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = f.place_name;
      b.addEventListener('click', () => {
        const [lng, lat] = f.center;
        marker?.remove();
        marker = L.circleMarker([lat, lng], { radius: 8, color: '#fff', weight: 2, fillColor: '#2346d8', fillOpacity: 1 }).addTo(map);
        f.bbox ? map.fitBounds([[f.bbox[1], f.bbox[0]], [f.bbox[3], f.bbox[2]]], { padding: [60, 60], maxZoom: 18 }) : map.flyTo([lat, lng], (f.properties.zoom ?? 14) + 1);
        liste.replaceChildren();
      });
      li.append(b); liste.append(li);
    }
  } catch (e) { liste.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

## Verwendete APIs

- [Geocoding](https://docs.mapx.at/de/api/geocoding.md) `GET api.mapx.at/v1/geocode`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

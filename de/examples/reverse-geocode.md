# Reverse Geocoding: Beispiel

Source: https://docs.mapx.at/de/examples/reverse-geocode/

> Karte anklicken und die nächste Adresse erhalten: Reverse Geocoding mit der mapx-Geocoding-API für Österreich, lauffähiger Code mit MapLibre GL JS.

Reverse Geocoding macht aus einer Koordinate die nächste Adresse, Straße oder den nächsten Ort. `types` begrenzt die Art des Ergebnisses, `radius` die Suchdistanz in Metern.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Reverse geocoding: click the map, get the nearest address, street or place.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 15 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const popup = new maplibregl.Popup({ closeButton: false });

map.on('click', async (e) => {
  const { lng, lat } = e.lngLat;
  try {
    const res = await fetch(`https://api.mapx.at/v1/reverse?lng=${lng}&lat=${lat}&limit=1&types=address,street,place`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const f = (await res.json()).features[0];
    popup.setLngLat(e.lngLat).setText(f ? [f.properties.name, f.properties.place_formatted].filter(Boolean).join(', ') : (de ? 'In der Nähe nichts gefunden' : 'Nothing found nearby')).addTo(map);
  } catch (err) { popup.setLngLat(e.lngLat).setText((de ? 'Fehler: ' : 'Error: ') + err.message).addTo(map); }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Reverse geocoding: click the map, get the nearest address, street or place.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const label = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
label.style.cssText = 'background:#fff;color:#000;padding:4px 8px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006';
const popup = new ol.Overlay({ element: label, positioning: 'bottom-center', offset: [0, -8] });
const map = new ol.Map({ target: 'map', layers: [base], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([16.3725, 48.2085]), zoom: 16 }) }); // Stephansplatz, Vienna

map.on('click', async (e) => {
  const [lng, lat] = ol.proj.toLonLat(e.coordinate);
  try {
    const res = await fetch(`https://api.mapx.at/v1/reverse?lng=${lng}&lat=${lat}&limit=1&types=address,street,place`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const f = (await res.json()).features[0];
    label.textContent = f ? [f.properties.name, f.properties.place_formatted].filter(Boolean).join(', ') : (de ? 'In der Nähe nichts gefunden' : 'Nothing found nearby');
  } catch (err) { label.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
  popup.setPosition(e.coordinate);
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Reverse geocoding: click the map, get the nearest address, street or place.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the addresses (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([48.2085, 16.3725], 16);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const popup = L.popup({ closeButton: false });
const show = (latlng, s) => popup.setLatLng(latlng).setContent(Object.assign(document.createElement('span'), { textContent: s })).openOn(map); // as text, not HTML
map.on('click', async (e) => {
  const { lng, lat } = e.latlng;
  try {
    const res = await fetch(`https://api.mapx.at/v1/reverse?lng=${lng}&lat=${lat}&limit=1&types=address,street,place`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const f = (await res.json()).features[0];
    show(e.latlng, f ? [f.properties.name, f.properties.place_formatted].filter(Boolean).join(', ') : (de ? 'In der Nähe nichts gefunden' : 'Nothing found nearby'));
  } catch (err) { show(e.latlng, (de ? 'Fehler: ' : 'Error: ') + err.message); }
});
````

## Verwendete APIs

- [Geocoding](https://docs.mapx.at/de/api/geocoding.md) `GET api.mapx.at/v1/reverse`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

# Verkehrsmeldungen und Sperren: Beispiel

Source: https://docs.mapx.at/de/examples/traffic/

> Verkehrsstil plus die aktiven Meldungen im sichtbaren Kartenausschnitt: Störungen und Sperren in Österreich mit der mapx-Verkehrs-API.

Der Stil `traffic-day` zeigt die aktuellen Meldungen und färbt auf Autobahnen die Abschnitte, auf denen Wechselverkehrszeichen das Tempo beschränken; eine flächendeckende Verkehrslage gibt es nicht. Meldungen (Unfälle, Baustellen, Sperren) kommen von `/v1/traffic/incidents`; das Beispiel lädt die aktiven für den sichtbaren Ausschnitt und zeichnet sie darüber. Eine Linie anklicken zeigt den Text.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Traffic: the traffic-day style (reports, sign speed limits on motorways) plus the current incident list for the visible area.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/traffic-day', center: [16.2, 48.1], zoom: 9 });
map.addControl(new maplibregl.NavigationControl());
const list = document.getElementById('meldungen');
const popup = new maplibregl.Popup();

map.on('load', () => {
  map.addSource('incidents', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'incidents', type: 'line', source: 'incidents', layout: { 'line-cap': 'round' }, paint: { 'line-color': ['case', ['get', 'closed'], '#c8372d', '#e8890c'], 'line-width': 6, 'line-opacity': 0.8 } });
  map.on('click', 'incidents', (e) => popup.setLngLat(e.lngLat).setText(e.features[0].properties.text).addTo(map));
  const load = async () => {
    const b = map.getBounds();
    const res = await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${b.getWest()},${b.getSouth()},${b.getEast()},${b.getNorth()}&active=true&limit=50`, AUTH);
    const fc = await res.json();
    map.getSource('incidents').setData(fc);
    const de = document.documentElement.lang.startsWith('de'); // page language: German or English text
    const first = fc.features[0] ? `${fc.features[0].properties.road ?? ''} ${fc.features[0].properties.type}` : '';
    list.textContent = de
      ? `${fc.features.length} aktive Meldung(en) im Ausschnitt` + (first ? `, z. B. ${first}` : '')
      : `${fc.features.length} active incident(s) in view` + (first ? `, e.g. ${first}` : '');
  };
  map.on('moveend', load);
  load();
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Traffic: the current incident list for the visible area on the street map (the traffic-day style has no raster tiles).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>; Verkehr/traffic: ASFINAG, EVIS.AT; <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const incidents = new ol.source.Vector();
const style = (f) => new ol.style.Style({ stroke: new ol.style.Stroke({ color: f.get('closed') ? 'rgba(200, 55, 45, 0.8)' : 'rgba(232, 137, 12, 0.8)', width: 6, lineCap: 'round' }) });
const label = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
label.style.cssText = 'background:#fff;color:#000;padding:4px 8px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006';
const popup = new ol.Overlay({ element: label, positioning: 'bottom-center', offset: [0, -8] });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: incidents, style })], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([16.2, 48.1]), zoom: 10 }) });
const geojson = new ol.format.GeoJSON({ featureProjection: 'EPSG:3857' }); // reads lon/lat GeoJSON into map coordinates
const list = document.getElementById('meldungen');

map.on('click', (e) => {
  const f = map.forEachFeatureAtPixel(e.pixel, (x) => x, { hitTolerance: 4 });
  label.textContent = f?.get('text') ?? '';
  popup.setPosition(f ? e.coordinate : undefined);
});
map.on('moveend', async () => {
  const [w, s, e, n] = ol.proj.transformExtent(map.getView().calculateExtent(map.getSize()), 'EPSG:3857', 'EPSG:4326');
  const de = document.documentElement.lang.startsWith('de'); // page language: German or English text
  try {
    const res = await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${w},${s},${e},${n}&active=true&limit=50`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const fc = await res.json();
    incidents.clear();
    incidents.addFeatures(geojson.readFeatures(fc));
    const first = fc.features[0] ? `${fc.features[0].properties.road ?? ''} ${fc.features[0].properties.type}` : '';
    list.textContent = de
      ? `${fc.features.length} aktive Meldung(en) im Ausschnitt` + (first ? `, z. B. ${first}` : '')
      : `${fc.features.length} active incident(s) in view` + (first ? `, e.g. ${first}` : '');
  } catch (err) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Traffic: the current incident list of the visible area as lines over the map, closures in red, a click shows the report.
// The traffic-day style (sign speed limits, traffic layers) has no raster tiles, so the streets style is the basemap here.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the traffic data (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, Verkehr: ASFINAG / EVIS.AT, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([48.1, 16.2], 10);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const list = document.getElementById('meldungen');
const popup = L.popup();
const incidents = L.geoJSON(null, {
  style: (f) => ({ color: f.properties.closed ? '#c8372d' : '#e8890c', weight: 6, opacity: 0.8, lineCap: 'round' }),
  // a map popup, not one bound to the line: it stays open when moveend reloads the incidents
  onEachFeature: (f, layer) => layer.on('click', (e) => popup.setLatLng(e.latlng).setContent(Object.assign(document.createElement('span'), { textContent: f.properties.text })).openOn(map)), // as text, not HTML
}).addTo(map);
async function load() {
  const b = map.getBounds();
  try {
    const res = await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${b.getWest()},${b.getSouth()},${b.getEast()},${b.getNorth()}&active=true&limit=50`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const fc = await res.json();
    incidents.clearLayers().addData(fc);
    const first = fc.features[0] ? `${fc.features[0].properties.road ?? ''} ${fc.features[0].properties.type}` : '';
    list.textContent = de
      ? `${fc.features.length} aktive Meldung(en) im Ausschnitt` + (first ? `, z. B. ${first}` : '')
      : `${fc.features.length} active incident(s) in view` + (first ? `, e.g. ${first}` : '');
  } catch (err) { list.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
map.on('moveend', load);
load();
````

Für `traffic-day` gibt es keine Rasterkacheln; die OpenLayers- und die Leaflet-Fassung zeigen die Meldungen deshalb über dem Stil `streets`, ohne die Tempo-Abschnitte.

## Verwendete APIs

- [Verkehr](https://docs.mapx.at/de/api/traffic.md) `GET api.mapx.at/v1/traffic/incidents`
- [Styles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/traffic-day`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

# Departures at a stop example

Source: https://docs.mapx.at/examples/departures/

> Click near a public transport stop in Austria and see the next departures: transit departures API with a live map and runnable code.

A click finds the nearest stop within 150 m via `/v1/stops` and lists, via `/v1/departures`, the next departures from the scheduled timetable (no real-time data). The dots are the stops of the visible area from `/v1/stops`.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Departures: click near a stop to see the next departures (/v1/stops finds the stop, /v1/departures lists the departures).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
const api = async (path) => { const r = await fetch('https://api.mapx.at/v1' + path, AUTH); if (!r.ok) throw new Error((await r.json()).title || r.statusText); return r.json(); };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 15.5 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const popup = new maplibregl.Popup({ maxWidth: '340px' });

// Show the stops of the visible area as dots.
map.on('load', () => {
  map.addSource('stops', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'stops', type: 'circle', source: 'stops', paint: { 'circle-radius': 6, 'circle-color': '#2346d8', 'circle-stroke-color': '#fff', 'circle-stroke-width': 2 } });
  const load = async () => {
    const b = map.getBounds();
    const { stops } = await api(`/stops?bbox=${[b.getWest(), b.getSouth(), b.getEast(), b.getNorth()].map((v) => v.toFixed(5)).join(',')}&limit=100`);
    map.getSource('stops').setData({ type: 'FeatureCollection', features: stops.map((s) => ({ type: 'Feature', properties: { name: s.name }, geometry: { type: 'Point', coordinates: s.location } })) });
  };
  map.on('moveend', load);
  load();
});

// A click finds the nearest stop within 150 m and lists its next 8 departures (timetable, no real-time data).
let seq = 0;
map.on('click', async (e) => {
  const n = ++seq, at = `${e.lngLat.lng.toFixed(6)},${e.lngLat.lat.toFixed(6)}`;
  popup.setLngLat(e.lngLat).setText(de ? 'Lade …' : 'Loading …').addTo(map);
  try {
    const stop = (await api(`/stops?near=${at}&radius=150&limit=1`)).stops[0];
    if (!stop) { popup.setText(de ? 'Keine Haltestelle in 150 m' : 'No stop within 150 m'); return; }
    const { departures } = await api(`/departures?stop=${encodeURIComponent(stop.id)}&limit=8`);
    if (n !== seq) return; // a newer click has taken over
    const el = document.createElement('div');
    el.append(Object.assign(document.createElement('strong'), { textContent: stop.name }));
    if (!departures.length) el.append(Object.assign(document.createElement('div'), { textContent: de ? 'Keine Abfahrten' : 'No departures' }));
    for (const d of departures) {
      const row = document.createElement('div');
      row.append(Object.assign(document.createElement('b'), { textContent: d.time.slice(11, 16) }), ` ${d.line.name} → ${d.headsign}`);
      el.append(row);
    }
    popup.setLngLat(stop.location).setDOMContent(el);
  } catch (err) { if (n === seq) popup.setText(err.message); }
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Departures: the stops of the visible area from /v1/stops; click a stop (or near one) for the next departures from /v1/departures.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>; Fahrplan/timetable: ÖBB-Personenverkehr AG, Mobilitätsverbünde Österreich OG; <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const stops = new ol.source.Vector();
const dot = new ol.style.Style({ image: new ol.style.Circle({ radius: 6, fill: new ol.style.Fill({ color: '#2346d8' }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const box = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
box.style.cssText = 'background:#fff;color:#000;padding:6px 10px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006;max-width:280px';
const popup = new ol.Overlay({ element: box, positioning: 'bottom-center', offset: [0, -10] });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: stops, style: dot })], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([16.3725, 48.2085]), zoom: 16.5 }) }); // Stephansplatz, Vienna

map.on('moveend', async () => {
  const [w, s, e, n] = ol.proj.transformExtent(map.getView().calculateExtent(map.getSize()), 'EPSG:3857', 'EPSG:4326');
  try {
    const res = await fetch(`https://api.mapx.at/v1/stops?bbox=${w},${s},${e},${n}&limit=100`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    stops.clear();
    stops.addFeatures((await res.json()).stops.map((st) => new ol.Feature({ geometry: new ol.geom.Point(ol.proj.fromLonLat(st.location)), id: st.id })));
  } catch (err) { console.warn(err); }
});

map.on('click', async (e) => {
  // a click on a dot asks for that stop, elsewhere for the stops within 150 m
  const hit = map.forEachFeatureAtPixel(e.pixel, (f) => f);
  const stop = hit ? hit.get('id') : ol.proj.toLonLat(e.coordinate).map((v) => v.toFixed(6)).join(',');
  const title = document.createElement('strong');
  box.replaceChildren(title);
  popup.setPosition(hit ? hit.getGeometry().getCoordinates() : e.coordinate);
  try {
    const res = await fetch(`https://api.mapx.at/v1/departures?stop=${encodeURIComponent(stop)}&limit=8&radius=150`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const { departures } = await res.json(); // scheduled timetable, no real-time data
    if (!departures.length) { title.textContent = de ? 'Keine Abfahrten in der Nähe' : 'No departures nearby'; return; }
    title.textContent = departures[0].stop.name;
    for (const d of departures) {
      const row = document.createElement('div');
      row.textContent = `${d.time.slice(11, 16)}  ${d.line.name}  ${de ? 'nach' : 'to'} ${d.headsign}`; // API texts as text, never as HTML
      box.append(row);
    }
  } catch (err) { title.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Departures: click near a stop to see the next departures (/v1/stops finds the stop, /v1/departures lists the departures).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the timetables, must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>; Fahrplan: ÖBB-Personenverkehr AG (CC BY 4.0), Mobilitätsverbünde Österreich OG';
const api = async (path) => { const r = await fetch('https://api.mapx.at/v1' + path, AUTH); if (!r.ok) throw new Error((await r.json()).title || r.statusText); return r.json(); };
const text = (s) => Object.assign(document.createElement('span'), { textContent: s }); // popup content as text, not HTML

const map = L.map('map').setView([48.2085, 16.3725], 16); // Wien, Stephansplatz
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const popup = L.popup({ maxWidth: 340 });

// Show the stops of the visible area as dots.
const dots = L.layerGroup().addTo(map);
async function load() {
  const b = map.getBounds();
  const { stops } = await api(`/stops?bbox=${[b.getWest(), b.getSouth(), b.getEast(), b.getNorth()].map((v) => v.toFixed(5)).join(',')}&limit=100`);
  dots.clearLayers();
  for (const s of stops) L.circleMarker([s.location[1], s.location[0]], { radius: 6, color: '#fff', weight: 2, fillColor: '#2346d8', fillOpacity: 1 }).addTo(dots);
}
map.on('moveend', load);
load();

// A click finds the nearest stop within 150 m and lists its next 8 departures (timetable, no real-time data).
let seq = 0;
map.on('click', async (e) => {
  const n = ++seq, at = `${e.latlng.lng.toFixed(6)},${e.latlng.lat.toFixed(6)}`;
  popup.setLatLng(e.latlng).setContent(text(de ? 'Lade …' : 'Loading …')).openOn(map);
  try {
    const stop = (await api(`/stops?near=${at}&radius=150&limit=1`)).stops[0];
    if (!stop) { popup.setContent(text(de ? 'Keine Haltestelle in 150 m' : 'No stop within 150 m')); return; }
    const { departures } = await api(`/departures?stop=${encodeURIComponent(stop.id)}&limit=8`);
    if (n !== seq) return; // a newer click has taken over
    const el = document.createElement('div');
    el.append(Object.assign(document.createElement('strong'), { textContent: stop.name }));
    if (!departures.length) el.append(Object.assign(document.createElement('div'), { textContent: de ? 'Keine Abfahrten' : 'No departures' }));
    for (const d of departures) {
      const row = document.createElement('div');
      row.append(Object.assign(document.createElement('b'), { textContent: d.time.slice(11, 16) }), ` ${d.line.name} → ${d.headsign}`);
      el.append(row);
    }
    popup.setLatLng([stop.location[1], stop.location[0]]).setContent(el);
  } catch (err) { if (n === seq) popup.setContent(text(err.message)); }
});
````

## APIs used

- [Transit](https://docs.mapx.at/api/transit.md) `GET api.mapx.at/v1/stops`
- [Transit](https://docs.mapx.at/api/transit.md) `GET api.mapx.at/v1/departures`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (Leaflet)

# Tourenplanung: Beispiel Stopp-Reihenfolge

Source: https://docs.mapx.at/de/examples/optimization/

> Haltestellen auf der Karte anklicken und die beste Reihenfolge ab Depot und zurück mit der mapx-Routenoptimierung erhalten: lauffähiges Beispiel.

Die Optimierung findet die beste Reihenfolge für eine Rundtour. Hier startet und endet das Fahrzeug in Melk. Bis zu neun Stopps anklicken, jeder Klick plant neu. Das Beispiel nutzt `GET /v1/optimized-trips/driving/{coordinates}` mit `source=first` und `roundtrip=true`; das geht mit dem öffentlichen Token, die Antwort enthält Reihenfolge und Strecke.

Für mehrere Fahrzeuge, Servicezeiten, Kapazitäten und Zeitfenster gibt es `POST /v1/optimization`. Das geht nur mit einem geheimen Token (`mx_sk_…`); ein öffentlicher Token bekommt `403 pk-not-allowed`. Diese Anfrage also über den eigenen Server schicken, nicht direkt aus der Webseite.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Tour planning: click stops (up to 9), the optimized-trips API finds the shortest order from the depot and back.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
// Note: the richer POST /v1/optimization (time windows, capacities) needs a secret token (mx_sk_…); call it from your server.
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const depot = [15.3294, 48.2272]; // Melk
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: depot, zoom: 9 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
const stops = [], markers = [];
let timer;
new maplibregl.Marker({ color: '#222' }).setLngLat(depot).addTo(map);

map.on('load', () => {
  map.addSource('tour', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'tour', type: 'line', source: 'tour', layout: { 'line-cap': 'round', 'line-join': 'round' }, paint: { 'line-color': '#2346d8', 'line-width': 5 } });
});
map.on('click', (e) => {
  if (stops.length >= 9) return;
  stops.push([e.lngLat.lng, e.lngLat.lat]);
  markers.push(new maplibregl.Marker({ color: '#c8372d' }).setLngLat(e.lngLat).addTo(map));
  clearTimeout(timer); // plan once after the last click
  if (stops.length >= 2) timer = setTimeout(plan, 600);
});
document.getElementById('reset').addEventListener('click', () => {
  stops.length = 0; markers.splice(0).forEach((m) => m.remove());
  map.getSource('tour').setData({ type: 'FeatureCollection', features: [] });
  document.getElementById('info').textContent = de ? 'Mindestens zwei Stopps anklicken.' : 'Click at least two stops.';
});

async function plan() {
  // Depot first, then the stops; source=first and roundtrip=true keep the depot as start and end.
  const coords = [depot, ...stops].map((p) => p.join(',')).join(';');
  try {
    const res = await fetch(`https://api.mapx.at/v1/optimized-trips/driving/${coords}?source=first&roundtrip=true&geometries=geojson`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const data = await res.json();
    const trip = data.trips[0];
    const order = data.waypoints.map((w, i) => ({ i, pos: w.waypoint_index })).filter((w) => w.i > 0).sort((a, b) => a.pos - b.pos).map((w) => w.i);
    map.getSource('tour').setData({ type: 'Feature', geometry: trip.geometry, properties: {} });
    document.getElementById('info').textContent = `${de ? 'Reihenfolge' : 'Order'}: ${order.join(' > ')}, ${(trip.distance / 1000).toFixed(0)} km, ${Math.round(trip.duration / 60)} min ${de ? 'Fahrzeit' : 'driving'}`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Tour planning: click stops (up to 9), the optimized-trips API finds the shortest order from the depot and back.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Note: the richer POST /v1/optimization (time windows, capacities) needs a secret token (mx_sk_…); call it from your server.
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const depot = [15.3294, 48.2272]; // Melk
const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const tour = new ol.source.Vector(), points = new ol.source.Vector();
const line = new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#2346d8', width: 5, lineCap: 'round', lineJoin: 'round' }) });
const pin = (color) => new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: tour, style: line }), new ol.layer.Vector({ source: points, style: pin('#c8372d') })], view: new ol.View({ center: ol.proj.fromLonLat(depot), zoom: 10 }) });
const geojson = new ol.format.GeoJSON({ featureProjection: 'EPSG:3857' }); // reads lon/lat GeoJSON into map coordinates
const depotMarker = new ol.Feature(new ol.geom.Point(ol.proj.fromLonLat(depot)));
depotMarker.setStyle(pin('#222'));
const stops = [];
let timer;
points.addFeature(depotMarker);

map.on('click', (e) => {
  if (stops.length >= 9) return;
  stops.push(ol.proj.toLonLat(e.coordinate));
  points.addFeature(new ol.Feature(new ol.geom.Point(e.coordinate)));
  clearTimeout(timer); // plan once after the last click
  if (stops.length >= 2) timer = setTimeout(plan, 600);
});
document.getElementById('reset').addEventListener('click', () => {
  stops.length = 0; points.clear(); points.addFeature(depotMarker); tour.clear();
  document.getElementById('info').textContent = de ? 'Mindestens zwei Stopps anklicken.' : 'Click at least two stops.';
});

async function plan() {
  // Depot first, then the stops; source=first and roundtrip=true keep the depot as start and end.
  const coords = [depot, ...stops].map((p) => p.join(',')).join(';');
  try {
    const res = await fetch(`https://api.mapx.at/v1/optimized-trips/driving/${coords}?source=first&roundtrip=true&geometries=geojson`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const data = await res.json();
    const trip = data.trips[0];
    const order = data.waypoints.map((w, i) => ({ i, pos: w.waypoint_index })).filter((w) => w.i > 0).sort((a, b) => a.pos - b.pos).map((w) => w.i);
    tour.clear();
    tour.addFeature(geojson.readFeature(trip.geometry));
    document.getElementById('info').textContent = `${de ? 'Reihenfolge' : 'Order'}: ${order.join(' > ')}, ${(trip.distance / 1000).toFixed(0)} km, ${Math.round(trip.duration / 60)} min ${de ? 'Fahrzeit' : 'driving'}`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Tour planning: click stops (up to 9), the optimized-trips API finds the shortest order from the depot and back.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the routing (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const depot = [15.3294, 48.2272]; // Melk (lng, lat)
const map = L.map('map').setView([48.12, 15.33], 10); // depot in the upper half, room for stops to the south
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const pin = (latlng, color) => L.circleMarker(latlng, { radius: 8, color: '#fff', weight: 2, fillColor: color, fillOpacity: 1 });
const tour = L.geoJSON(null, { style: { color: '#2346d8', weight: 5, opacity: 1 } }).addTo(map);
const markers = L.layerGroup().addTo(map);
pin([depot[1], depot[0]], '#222').addTo(map);
const stops = [];
let timer;
map.on('click', (e) => {
  if (stops.length >= 9) return;
  stops.push([e.latlng.lng, e.latlng.lat]);
  pin(e.latlng, '#c8372d').addTo(markers);
  clearTimeout(timer); // plan once after the last click
  if (stops.length >= 2) timer = setTimeout(plan, 600);
});
document.getElementById('reset').addEventListener('click', () => {
  stops.length = 0; markers.clearLayers(); tour.clearLayers();
  document.getElementById('info').textContent = de ? 'Mindestens zwei Stopps anklicken.' : 'Click at least two stops.';
});
async function plan() {
  // Depot first, then the stops; source=first and roundtrip=true keep the depot as start and end.
  const coords = [depot, ...stops].map((p) => p.join(',')).join(';');
  try {
    const res = await fetch(`https://api.mapx.at/v1/optimized-trips/driving/${coords}?source=first&roundtrip=true&geometries=geojson`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const data = await res.json();
    const trip = data.trips[0];
    const order = data.waypoints.map((w, i) => ({ i, pos: w.waypoint_index })).filter((w) => w.i > 0).sort((a, b) => a.pos - b.pos).map((w) => w.i);
    tour.clearLayers().addData(trip.geometry);
    document.getElementById('info').textContent = `${de ? 'Reihenfolge' : 'Order'}: ${order.join(' > ')}, ${(trip.distance / 1000).toFixed(0)} km, ${Math.round(trip.duration / 60)} min ${de ? 'Fahrzeit' : 'driving'}`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

## Verwendete APIs

- [Optimization](https://docs.mapx.at/de/api/optimization.md) `GET api.mapx.at/v1/optimized-trips/{profile}/{coordinates}`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

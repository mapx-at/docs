# Tourenplanung: Beispiel Stopp-Reihenfolge

Source: https://docs.mapx.at/de/examples/optimization/

> Haltestellen auf der Karte anklicken und die beste Reihenfolge ab Depot und zurück mit der mapx-Routenoptimierung erhalten: lauffähiges Beispiel.

Die Optimization-API ordnet Stopps für ein oder mehrere Fahrzeuge, mit Servicezeiten, Kapazitäten und Zeitfenstern. Hier startet und endet ein Fahrzeug in Melk. Bis zu neun Stopps anklicken, jeder Klick plant neu. Die Reihenfolge wird mit einer Directions-Anfrage durch alle Stopps gezeichnet.

`POST /v1/optimization` geht nur mit einem geheimen Token (`mx_sk_…`); ein öffentlicher Token bekommt `403 pk-not-allowed`. In der eigenen Anwendung die Anfrage daher über den eigenen Server schicken, nicht direkt aus der Webseite.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Tour planning: click stops (up to 9), the optimization API finds the shortest order from the depot and back.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
// Note: POST needs a secret token (mx_sk_…), public tokens are read-only. Run this call from your server, not from a web page.
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const depot = [15.3294, 48.2272]; // Melk
const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: depot, zoom: 9 });
map.addControl(new maplibregl.NavigationControl());
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
  const body = { vehicles: [{ id: 1, start: depot, end: depot }], services: stops.map((location, i) => ({ id: i + 1, location, service: 300 })) };
  try {
    let res = await fetch('https://api.mapx.at/v1/optimization', { method: 'POST', headers: { ...AUTH.headers, 'content-type': 'application/json' }, body: JSON.stringify(body) });
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const tour = (await res.json()).routes[0];
    const order = tour.steps.map((s) => s.location);
    // The tour only lists the order; draw the roads with one directions request through all stops.
    res = await fetch(`https://api.mapx.at/v1/directions/driving/${order.map((p) => p.join(',')).join(';')}?overview=full&geometries=geojson`, AUTH);
    const route = (await res.json()).routes[0];
    map.getSource('tour').setData({ type: 'Feature', geometry: route.geometry, properties: {} });
    document.getElementById('info').textContent = `${de ? 'Reihenfolge' : 'Order'}: ${tour.steps.filter((s) => s.type === 'job').map((s) => s.id).join(' > ')}, ${(tour.distance / 1000).toFixed(0)} km, ${Math.round(tour.duration / 60)} min ${de ? 'Fahrzeit' : 'driving'}`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Tour planning: click stops (up to 9), the optimization API finds the shortest order from the depot and back.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Note: POST needs a secret token (mx_sk_…), public tokens are read-only. Run this call from your server, not from a web page.
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

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
  const body = { vehicles: [{ id: 1, start: depot, end: depot }], services: stops.map((location, i) => ({ id: i + 1, location, service: 300 })) };
  try {
    let res = await fetch('https://api.mapx.at/v1/optimization', { method: 'POST', headers: { ...AUTH.headers, 'content-type': 'application/json' }, body: JSON.stringify(body) });
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const plan = (await res.json()).routes[0];
    const order = plan.steps.map((s) => s.location);
    // The tour only lists the order; draw the roads with one directions request through all stops.
    res = await fetch(`https://api.mapx.at/v1/directions/driving/${order.map((p) => p.join(',')).join(';')}?overview=full&geometries=geojson`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    tour.clear();
    tour.addFeature(geojson.readFeature((await res.json()).routes[0].geometry));
    document.getElementById('info').textContent = `${de ? 'Reihenfolge' : 'Order'}: ${plan.steps.filter((s) => s.type === 'job').map((s) => s.id).join(' > ')}, ${(plan.distance / 1000).toFixed(0)} km, ${Math.round(plan.duration / 60)} min ${de ? 'Fahrzeit' : 'driving'}`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Tour planning: click stops (up to 9), the optimization API finds the shortest order from the depot and back.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style and the routing (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

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
  const body = { vehicles: [{ id: 1, start: depot, end: depot }], services: stops.map((location, i) => ({ id: i + 1, location, service: 300 })) };
  try {
    let res = await fetch('https://api.mapx.at/v1/optimization', { method: 'POST', headers: { ...AUTH.headers, 'content-type': 'application/json' }, body: JSON.stringify(body) });
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const plan = (await res.json()).routes[0];
    const order = plan.steps.map((s) => s.location);
    // The tour only lists the order; draw the roads with one directions request through all stops.
    res = await fetch(`https://api.mapx.at/v1/directions/driving/${order.map((p) => p.join(',')).join(';')}?overview=full&geometries=geojson`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    tour.clearLayers().addData((await res.json()).routes[0].geometry);
    document.getElementById('info').textContent = `${de ? 'Reihenfolge' : 'Order'}: ${plan.steps.filter((s) => s.type === 'job').map((s) => s.id).join(' > ')}, ${(plan.distance / 1000).toFixed(0)} km, ${Math.round(plan.duration / 60)} min ${de ? 'Fahrzeit' : 'driving'}`;
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
````

## Verwendete APIs

- [Optimization](https://docs.mapx.at/de/api/optimization.md) `POST api.mapx.at/v1/optimization`
- [Directions](https://docs.mapx.at/de/api/directions.md) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

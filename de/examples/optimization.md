# Tourenplanung: Beispiel Stopp-Reihenfolge

Source: https://docs.mapx.at/de/examples/optimization/

> Haltestellen auf der Karte anklicken und die beste Reihenfolge ab Depot und zurück mit der mapx-Routenoptimierung erhalten: lauffähiges Beispiel.

Die Optimization-API ordnet Stopps für ein oder mehrere Fahrzeuge, mit Servicezeiten, Kapazitäten und Zeitfenstern. Hier startet und endet ein Fahrzeug in Melk. Bis zu neun Stopps anklicken, jeder Klick plant neu. Die Reihenfolge wird mit einer Directions-Anfrage durch alle Stopps gezeichnet.

`POST /v1/optimization` geht nur mit einem geheimen Token (`mx_sk_…`); ein öffentlicher Token bekommt `403 pk-not-allowed`. In der eigenen Anwendung die Anfrage daher über den eigenen Server schicken, nicht direkt aus der Webseite.

## Code

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

## Verwendete APIs

- [Optimization](https://docs.mapx.at/de/api/optimization.md) `POST api.mapx.at/v1/optimization`
- [Directions](https://docs.mapx.at/de/api/directions.md) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`

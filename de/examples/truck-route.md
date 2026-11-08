# Lkw-Route mit Optionen

Fahrzeugmaße, Maut und Autobahn meiden, Alternativen.

Das Profil `driving-truck` mit Gewicht und Höhe wählen: Straßen mit passenden Verboten und Beschränkungen werden gemieden, die Antwort nennt die GO-mautpflichtigen Kilometer. Für Pkw (`driving`) lassen sich Maut und Autobahn meiden, dazu gibt es bis zu zwei Alternativen. Start und Ziel anklicken.

```js
// Truck route: start and destination by click, vehicle size, avoid tolls and motorways, alternatives in grey.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.4, 48.2], zoom: 8 });
map.addControl(new maplibregl.NavigationControl());
const $ = (id) => document.getElementById(id);
const pts = [], markers = [];

map.on('load', () => {
  map.addSource('routes', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'routes', type: 'line', source: 'routes', layout: { 'line-cap': 'round' }, paint: { 'line-color': ['case', ['==', ['get', 'main'], 1], '#2346d8', '#9aa4b8'], 'line-width': ['case', ['==', ['get', 'main'], 1], 6, 4] } });
});
map.on('click', (e) => {
  if (pts.length === 2) { pts.length = 0; markers.splice(0).forEach((m) => m.remove()); }
  pts.push([e.lngLat.lng, e.lngLat.lat]);
  markers.push(new maplibregl.Marker({ color: pts.length === 1 ? '#1d7f4e' : '#c8372d' }).setLngLat(e.lngLat).addTo(map));
  if (pts.length === 2) route();
});
for (const id of ['profile', 'toll', 'motorway', 'weight', 'height']) $(id).addEventListener('change', () => pts.length === 2 && route());

async function route() {
  const profile = $('profile').value, q = new URLSearchParams({ overview: 'full', geometries: 'geojson', alternatives: 'true' });
  if (profile === 'driving-truck') { q.set('weight', $('weight').value); q.set('height', $('height').value); q.delete('alternatives'); }
  const exclude = ['toll', 'motorway'].filter((k) => $(k).checked);
  if (profile === 'driving' && exclude.length) q.set('exclude', exclude.join(','));
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/${profile}/${pts.map((p) => p.join(',')).join(';')}?${q}`, AUTH);
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const { routes } = await res.json();
    map.getSource('routes').setData({ type: 'FeatureCollection', features: routes.map((r, i) => ({ type: 'Feature', properties: { main: i === 0 ? 1 : 0 }, geometry: r.geometry })).reverse() });
    const r = routes[0];
    $('info').textContent = `${(r.distance / 1000).toFixed(1)} km, ${Math.round(r.duration / 60)} min` + (r.toll?.truck_go_km != null ? `, ${r.toll.truck_go_km.toFixed(1)} km subject to GO toll` : '') + (routes.length > 1 ? `, ${routes.length - 1} alternative(s)` : '');
  } catch (err) { $('info').textContent = 'Error: ' + err.message; }
}
```

## Verwendete APIs

- [Directions](/de/api/directions/) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
# Public transit route

Click start and destination, get walking and riding legs.

The transit API combines walks on the road network with rides from the timetable. Without `depart_at` it plans for now; `arrive_by` plans backwards. Walking legs are drawn dashed, rides solid.

```js
// Public transit: click start and destination, get walking and riding legs for the next departure.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
const pts = [], markers = [];

map.on('load', () => {
  map.addSource('legs', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  const line = { 'line-width': 5 };
  map.addLayer({ id: 'legs-ride', type: 'line', source: 'legs', filter: ['!=', ['get', 'mode'], 'walking'], layout: { 'line-cap': 'round', 'line-join': 'round' }, paint: { ...line, 'line-color': '#2346d8' } });
  map.addLayer({ id: 'legs-walk', type: 'line', source: 'legs', filter: ['==', ['get', 'mode'], 'walking'], layout: { 'line-join': 'round' }, paint: { ...line, 'line-color': '#667', 'line-dasharray': [1, 1.5] } });
});
map.on('click', async (e) => {
  if (pts.length === 2) { pts.length = 0; markers.splice(0).forEach((m) => m.remove()); }
  pts.push([e.lngLat.lng, e.lngLat.lat]);
  markers.push(new maplibregl.Marker({ color: pts.length === 1 ? '#1d7f4e' : '#c8372d' }).setLngLat(e.lngLat).addTo(map));
  if (pts.length < 2) return;
  const info = document.getElementById('info');
  try {
    const res = await fetch(`https://api.mapx.at/v1/directions/transit/${pts.map((p) => p.join(',')).join(';')}?routes=1`, AUTH); // add depart_at=2026-11-09T08:00 for a fixed time
    if (!res.ok) throw new Error((await res.json()).detail ?? 'HTTP ' + res.status);
    const route = (await res.json()).routes[0];
    map.getSource('legs').setData({ type: 'FeatureCollection', features: route.legs.map((l) => ({ type: 'Feature', properties: { mode: l.mode }, geometry: l.geometry })) });
    info.textContent = `${route.departure.slice(11, 16)} to ${route.arrival.slice(11, 16)}, ${Math.round(route.duration / 60)} min, ${route.transfers} transfer(s): ` + route.legs.map((l) => l.summary).join(' > ');
  } catch (err) { info.textContent = 'Error: ' + err.message; }
});
```

## APIs used

- [Transit](/api/transit/) `GET api.mapx.at/v1/directions/transit/{coordinates}`
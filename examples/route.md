# Route

Click start and destination, get the way with length and duration.

The routing API works on the GIP.at network for walking, cycling, mountain biking and driving. The response has the Mapbox shape `routes[0]` with `geometry` (GeoJSON line), `distance` in metres and `duration` in seconds.

```js
// Route: click start and destination on the map, the Directions API returns a GeoJSON line.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.33, 48.23], zoom: 13 }); // Melk
map.addControl(new maplibregl.NavigationControl());
let a = null, b = null, markerA, markerB;

map.on('load', () => {
  map.addSource('route', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'route', type: 'line', source: 'route', layout: { 'line-cap': 'round', 'line-join': 'round' }, paint: { 'line-color': '#2346d8', 'line-width': 5 } });
});
map.on('click', async (e) => {
  if (a && b) { a = b = null; markerA?.remove(); markerB?.remove(); map.getSource('route').setData({ type: 'FeatureCollection', features: [] }); }
  const p = [e.lngLat.lng, e.lngLat.lat];
  if (!a) { a = p; markerA = new maplibregl.Marker({ color: '#1d7f4e' }).setLngLat(p).addTo(map); return; }
  b = p; markerB = new maplibregl.Marker({ color: '#c8372d' }).setLngLat(p).addTo(map);
  const profil = document.getElementById('profil').value; // walking, cycling, cycling-mtb or driving
  const url = `https://api.mapx.at/v1/directions/${profil}/${a.join(',')};${b.join(',')}?geometries=geojson&overview=full`;
  const info = document.getElementById('info');
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('no route (HTTP ' + r.status + ')');
    const route = (await r.json()).routes[0]; // Mapbox shape: routes[0].geometry is a GeoJSON line
    map.getSource('route').setData({ type: 'Feature', geometry: route.geometry, properties: {} });
    info.textContent = `${(route.distance / 1000).toFixed(2)} km, ${Math.round(route.duration / 60)} min`;
  } catch (err) { info.textContent = 'Error: ' + err.message; }
});
```

## APIs used

- [Routing](/api/directions/#routing) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
# Route

Start und Ziel klicken, Weg samt Länge und Dauer erhalten.

Die Routing-API arbeitet auf dem GIP.at-Netz für Fuß, Rad und Auto. Die Antwort hat die Mapbox-Form `routes[0]` mit `geometry` (GeoJSON-Linie), `distance` in Metern und `duration` in Sekunden.

```js
// Route: Start und Ziel in die Karte klicken, die Directions-API liefert eine GeoJSON-Linie.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/karte', center: [15.33, 48.23], zoom: 13 }); // Melk
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
  const profil = document.getElementById('profil').value; // walking, cycling oder driving
  const url = `https://api.mapx.at/v1/directions/${profil}/${a.join(',')};${b.join(',')}?geometries=geojson&overview=full`;
  const info = document.getElementById('info');
  try {
    const r = await fetch(url);
    if (!r.ok) throw new Error('keine Route (HTTP ' + r.status + ')');
    const route = (await r.json()).routes[0]; // Mapbox-Form: routes[0].geometry ist eine GeoJSON-Linie
    map.getSource('route').setData({ type: 'Feature', geometry: route.geometry, properties: {} });
    info.textContent = `${(route.distance / 1000).toFixed(2)} km, ${Math.round(route.duration / 60)} min`;
  } catch (err) { info.textContent = 'Fehler: ' + err.message; }
});
```

## Verwendete APIs

- [Routing](/api/routing/#directions) `GET api.mapx.at/v1/directions/{profile}/{coordinates}`
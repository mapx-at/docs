# Route

Start und Ziel klicken, Weg samt Länge und Dauer erhalten.

Die Routing-API arbeitet auf dem GIP.at-Netz für Fuß, Rad und Auto. Die Antwort ist ein GeoJSON-Feature mit `distance_m` und `duration_s`.

> Such- und Routing-Dienst antworten nur auf freigegebene Herkünfte (CORS). Die Beispiele laufen hier, sobald `docs.mapx.at` freigegeben ist; auf eigenen Seiten braucht es ebenfalls eine Freigabe der Herkunft.

```js
// Route: Start und Ziel in die Karte klicken, die Routing-API liefert eine GeoJSON-Linie.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://mapx.at/styles/standard-tag-normal.json', center: [15.33, 48.23], zoom: 13 }); // Melk
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
  const profil = document.getElementById('profil').value; // foot, bike oder car
  const url = `https://routing.mapx.at/route?profile=${profil}&from=${a.join(',')}&to=${b.join(',')}`; // künftig /v1/directions
  const info = document.getElementById('info');
  try {
    const r = await fetch(url);
    if (!r.ok) throw new Error('keine Route (HTTP ' + r.status + ')');
    const f = await r.json();
    map.getSource('route').setData(f);
    info.textContent = `${(f.properties.distance_m / 1000).toFixed(2)} km, ${Math.round(f.properties.duration_s / 60)} min`;
  } catch (err) { info.textContent = 'Fehler: ' + err.message; }
});
```

## Verwendete APIs

- Routing `GET routing.mapx.at/route`

> Aktuell `routing.mapx.at/route`; die Mapbox-ähnliche Form `GET /v1/directions` ist geplant.
# Erreichbarkeit (Isochronen)


Ein Klick setzt den Startpunkt. Die Isochronen-API liefert je Stufe ein MultiPolygon, die größte Fläche zuerst.

```js
// Isochrones: areas reachable in 5, 10 and 15 minutes.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.33, 48.23], zoom: 13 });
map.addControl(new maplibregl.NavigationControl());

map.on('load', () => {
  map.addSource('iso', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'iso-flaeche', type: 'fill', source: 'iso', paint: { 'fill-color': '#2346d8', 'fill-opacity': 0.12 } });
  map.addLayer({ id: 'iso-linie', type: 'line', source: 'iso', paint: { 'line-color': '#2346d8', 'line-width': 2 } });
  abfragen([15.33, 48.23]);
});
map.on('click', (e) => abfragen([e.lngLat.lng, e.lngLat.lat]));

async function abfragen(ort) {
  const profil = document.getElementById('profil').value; // walking, cycling or driving
  const url = `https://api.mapx.at/v1/isochrone/${profil}/${ort.join(',')}?contours_minutes=5,10,15`;
  try {
    const r = await fetch(url, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    map.getSource('iso').setData(await r.json()); // one MultiPolygon per step, largest first
  } catch (err) { document.getElementById('info').textContent = 'Error: ' + err.message; }
}
```

## Verwendete APIs

- [Routing](/de/api/directions/#isochrones) `GET api.mapx.at/v1/isochrone/{profile}/{coordinate}`


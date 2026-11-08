# Höhe per Klick


Die Höhen-API liefert GeoJSON mit `elevation` in Metern. DTM zeigt das Gelände, DSM Gebäude und Bewuchs.

```js
// Query elevation: a click asks /v1/elevation (DTM or DSM) for the point.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/outdoors', center: [12.694, 47.074], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
const popup = new maplibregl.Popup({ closeButton: false });

map.on('click', async (e) => {
  const modell = document.getElementById('modell').value; // dtm or dsm
  const url = `https://api.mapx.at/v1/elevation?points=${e.lngLat.lng.toFixed(6)},${e.lngLat.lat.toFixed(6)}&model=${modell}`;
  try {
    const { features } = await (await fetch(url, AUTH)).json();
    const h = features[0].properties.elevation;
    popup.setLngLat(e.lngLat).setText(h == null ? 'Outside the data' : `${h.toLocaleString('de-AT')} m (${modell.toUpperCase()})`).addTo(map);
  } catch (err) {
    popup.setLngLat(e.lngLat).setText('Error: ' + err.message).addTo(map);
  }
});
```

## Verwendete APIs

- [Höhe](/de/api/elevation/) `GET /v1/elevation`


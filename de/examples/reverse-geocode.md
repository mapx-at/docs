# Reverse Geocoding


Reverse Geocoding macht aus einer Koordinate die nächste Adresse, Straße oder den nächsten Ort. `types` begrenzt die Art des Ergebnisses, `radius` die Suchdistanz in Metern.

```js
// Reverse geocoding: click the map, get the nearest address, street or place.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 15 });
map.addControl(new maplibregl.NavigationControl());
const popup = new maplibregl.Popup({ closeButton: false });

map.on('click', async (e) => {
  const { lng, lat } = e.lngLat;
  try {
    const res = await fetch(`https://api.mapx.at/v1/reverse?lng=${lng}&lat=${lat}&limit=1&types=address,street,place`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const f = (await res.json()).features[0];
    popup.setLngLat(e.lngLat).setText(f ? f.place_name : 'Nothing found nearby').addTo(map);
  } catch (err) { popup.setLngLat(e.lngLat).setText('Error: ' + err.message).addTo(map); }
});
```

## Verwendete APIs

- [Geocoding](/de/api/geocoding/) `GET api.mapx.at/v1/reverse`


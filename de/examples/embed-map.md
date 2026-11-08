# Karte einbetten


Der kürzeste Weg zu einer Karte: MapLibre GL JS, das pmtiles-Protokoll und die Stil-URL. Zoom, Neigung und Maßstab kommen als Steuerelemente dazu.

```js
// Embed a map: the ready-made mapx style, one call.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({
  transformRequest,
  container: 'map',
  style: 'https://api.mapx.at/v1/styles/streets',
  center: [13.04, 47.80], // Salzburg
  zoom: 12,
});
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.ScaleControl());
```

## Verwendete APIs

- [Styles](/de/api/styles/) `GET api.mapx.at/v1/styles/streets`


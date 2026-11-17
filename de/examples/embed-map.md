# Karte in eine HTML-Seite einbetten

Source: https://docs.mapx.at/de/examples/embed-map/

> Eine mapx-Karte mit dem fertigen Stil per MapLibre GL JS in eine beliebige HTML-Seite einbetten: minimales, lauffähiges Beispiel mit öffentlichem Token.

Der kürzeste Weg zu einer Karte: MapLibre GL JS, das pmtiles-Protokoll und die Stil-URL. Zoom, Neigung und Maßstab kommen als Steuerelemente dazu.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

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
````

## Verwendete APIs

- [Styles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/streets`

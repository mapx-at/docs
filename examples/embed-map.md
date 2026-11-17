# Embed a map in an HTML page

Source: https://docs.mapx.at/examples/embed-map/

> Embed a mapx map with the ready-made style in any HTML page with MapLibre GL JS: a minimal runnable example with a public token.

The shortest way to a map: MapLibre GL JS, the pmtiles protocol and the style URL. Zoom, tilt and scale are added as controls.

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

## APIs used

- [Styles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/streets`

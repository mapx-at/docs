# Embed a map

A map with the ready-made mapx style in an HTML page.

The shortest way to a map: MapLibre GL JS, the pmtiles protocol and the style URL. Zoom, tilt and scale are added as controls.

```js
// Karte einbetten: der fertige Stil von mapx, ein Aufruf.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({
  container: 'map',
  style: 'https://api.mapx.at/v1/styles/streets',
  center: [13.04, 47.80], // Salzburg
  zoom: 12,
});
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.ScaleControl());
```

## APIs used

- [Styles](/en/api/styles/) `GET api.mapx.at/v1/styles/streets`
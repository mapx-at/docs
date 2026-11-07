# Karte einbetten

Eine Karte mit dem fertigen mapx-Stil in einer HTML-Seite.

Der kürzeste Weg zu einer Karte: MapLibre GL JS, das pmtiles-Protokoll und die Stil-URL. Zoom, Neigung und Maßstab kommen als Steuerelemente dazu.

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

## Verwendete APIs

- [Styles](/api/styles/) `GET api.mapx.at/v1/styles/streets`
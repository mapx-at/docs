# TileJSON-Quellen

Orthophoto und Höhenlinien ohne fertigen Stil einbinden.

Eine TileJSON-URL pro Produkt genügt: MapLibre liest Kachel-URL, Zoombereich und Attribution daraus. Hier ein eigener, minimaler Stil.

```js
// TileJSON einbinden: Orthophoto und Höhenlinien als Quellen ohne Dateinamen und ohne Stil.
const map = new maplibregl.Map({
  container: 'map', center: [11.4, 47.27], zoom: 12.5, // Innsbruck
  style: {
    version: 8,
    sources: {
      ortho: { type: 'raster', url: 'https://api.mapx.at/v1/tiles/orthophoto.json', tileSize: 512 },
      linien: { type: 'vector', url: 'https://api.mapx.at/v1/tiles/contours.json' },
    },
    layers: [
      { id: 'ortho', type: 'raster', source: 'ortho' },
      { id: 'linien', type: 'line', source: 'linien', 'source-layer': 'contour_l', paint: { 'line-color': '#ffd24a', 'line-width': 0.8, 'line-opacity': 0.9 } },
    ],
  },
});
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
```

## Verwendete APIs

- [TileJSON](/api/tilejson/) `dop-ortho`, `hoehenlinien`
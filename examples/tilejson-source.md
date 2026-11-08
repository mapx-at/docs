# TileJSON sources

Add orthophoto and contours without a ready-made style.

One TileJSON URL per product is enough: MapLibre reads tile URL, zoom range and attribution from it. Here a custom, minimal style.

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

## APIs used

- [TileJSON](/api/tiles/) `orthophoto`, `contours`
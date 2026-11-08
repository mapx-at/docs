# 3D terrain

Aerial imagery over the terrain model, tilted and exaggerated.

The Terrain-RGB tileset via TileJSON as a `raster-dem` source, `map.setTerrain()` and a slider for the exaggeration.

```js
// 3D-Gelände: Terrain-RGB-Kacheln von mapx per TileJSON als Höhenquelle.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({
  container: 'map',
  style: 'https://api.mapx.at/v1/styles/satellite', // Luftbild, wird über das Gelände gelegt
  center: [12.694, 47.074], // Großglockner
  zoom: 12.5, pitch: 70, bearing: 20, maxPitch: 80,
});
map.addControl(new maplibregl.NavigationControl({ visualizePitch: true }));

map.on('load', () => {
  map.addSource('hoehe', { type: 'raster-dem', url: 'https://api.mapx.at/v1/tiles/dtm.json', tileSize: 512, encoding: 'mapbox' });
  map.setTerrain({ source: 'hoehe', exaggeration: 1.4 });
});

document.getElementById('ueberhoehung').addEventListener('input', (e) => {
  map.setTerrain({ source: 'hoehe', exaggeration: Number(e.target.value) });
});
```

## APIs used

- [TileJSON](/api/tiles/) `dtm`
- Style `satellite`
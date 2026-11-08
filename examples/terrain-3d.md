# 3D terrain

Aerial imagery over the terrain model, tilted and exaggerated.

The Terrain-RGB tileset via TileJSON as a `raster-dem` source, `map.setTerrain()` and a slider for the exaggeration.

```js
// 3D terrain: mapx Terrain-RGB tiles via TileJSON as elevation source.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({
  transformRequest,
  container: 'map',
  style: 'https://api.mapx.at/v1/styles/satellite', // aerial imagery, draped over the terrain
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
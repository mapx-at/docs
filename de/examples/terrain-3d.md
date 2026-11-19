# 3D-Gelände: Beispiel mit Orthophoto

Source: https://docs.mapx.at/de/examples/terrain-3d/

> Luftbild über dem Geländemodell, geneigt und überhöht: 3D-Gelände in MapLibre GL JS mit mapx-Terrain-RGB-Kacheln, lauffähiger Code.

Das Terrain-RGB-Tileset per TileJSON als `raster-dem`-Quelle, `map.setTerrain()` und ein Regler für die Überhöhung.

## Code

3D-Gelände gibt es nur in MapLibre; OpenLayers und Leaflet zeichnen flache Karten.

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// 3D terrain: mapx Terrain-RGB tiles via TileJSON as elevation source.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({
  attributionControl: false,
  transformRequest,
  container: 'map',
  style: 'https://api.mapx.at/v1/styles/satellite', // aerial imagery, draped over the terrain
  center: [12.694, 47.074], // Großglockner
  zoom: 12.5, pitch: 70, bearing: 20, maxPitch: 80,
});
map.addControl(new maplibregl.NavigationControl({ visualizePitch: true }));
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

map.on('load', () => {
  map.addSource('hoehe', { type: 'raster-dem', url: 'https://api.mapx.at/v1/tiles/dtm.json', tileSize: 512, encoding: 'mapbox' });
  map.setTerrain({ source: 'hoehe', exaggeration: 1.4 });
});

document.getElementById('ueberhoehung').addEventListener('input', (e) => {
  map.setTerrain({ source: 'hoehe', exaggeration: Number(e.target.value) });
});
````

## Verwendete APIs

- [TileJSON](https://docs.mapx.at/de/api/tiles.md) `dtm`
- Stil `satellite`

# 3D-Gebäude

Gebäude mit BEV-Höhen als 3D-Körper, Kamera geneigt.

Der Stil `streets` enthält eine extrudierte Gebäudeebene, die standardmäßig aus ist. Die Ebenen-IDs stehen in den Style-Metadaten (`mapx:groups`), damit lässt sich eine ganze Gruppe einschalten, ohne die Namen zu kennen. Auf Zoom 15 oder höher gehen und die Karte kippen (rechte Maustaste oder zwei Finger).

```js
// 3D buildings: switch on the extruded building layer of the streets style and tilt the camera.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 16.2, pitch: 62, bearing: -20 }); // Stephansplatz, Vienna
map.addControl(new maplibregl.NavigationControl());

map.on('style.load', () => {
  // The style lists its layer groups in the metadata, so no layer names are hard-coded.
  for (const id of map.getStyle().metadata['mapx:groups'].buildings3d) map.setLayoutProperty(id, 'visibility', 'visible');
});
```

## Verwendete APIs

- [Styles](/de/api/styles/) `GET api.mapx.at/v1/styles/{id}`
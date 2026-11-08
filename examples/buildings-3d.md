# 3D buildings

Extruded buildings with BEV heights, camera tilted.

The `streets` style contains an extruded building layer that is switched off by default. The layer ids are listed in the style metadata (`mapx:groups`), so you switch a whole group on without knowing the names. Zoom in to level 15 or more and tilt the map (right mouse button or two fingers).

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

## APIs used

- [Styles](/api/styles/) `GET api.mapx.at/v1/styles/{id}`
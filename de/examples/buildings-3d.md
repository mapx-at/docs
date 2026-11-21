# 3D-Gebäude: Beispiel

Source: https://docs.mapx.at/de/examples/buildings-3d/

> Extrudierte 3D-Gebäude mit BEV-Höhen auf der mapx-Vektor-Basiskarte, Kamera geneigt, in MapLibre GL JS. Lauffähiger Code zum Kopieren.

Der Stil `streets` enthält eine extrudierte Gebäudeebene, die standardmäßig aus ist. Die Ebenen-IDs stehen in den Stil-Metadaten (`mapx:groups`), damit lässt sich eine ganze Gruppe einschalten, ohne die Namen zu kennen. Auf Zoom 15 oder höher gehen und die Karte kippen (rechte Maustaste oder zwei Finger).

## Code

3D-Gebäude gibt es nur in MapLibre; OpenLayers und Leaflet zeichnen flache Karten.

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// 3D buildings: switch on the extruded building layer of the streets style and tilt the camera.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 16.2, pitch: 62, bearing: -20 }); // Stephansplatz, Vienna
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

map.on('style.load', () => {
  // The style lists its layer groups in the metadata, so no layer names are hard-coded.
  for (const id of map.getStyle().metadata['mapx:groups'].buildings3d) map.setLayoutProperty(id, 'visibility', 'visible');
});
````

## Verwendete APIs

- [Styles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}`

# Cadastre example: parcel on click

Source: https://docs.mapx.at/examples/parcel-on-click/

> Show parcel boundaries (Kataster) over aerial imagery; a click names the parcel number. Cadastre tileset and parcel API, runnable code.

The cadastre tileset contains boundary lines from zoom 16 with parcel number and cadastral municipality. `queryRenderedFeatures` reads them at the click point.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Cadastre: a click on a parcel boundary shows the parcel number and cadastral municipality.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/satellite', center: [15.3306, 48.2275], zoom: 18 }); // Melk
map.addControl(new maplibregl.NavigationControl());

map.on('load', () => {
  // Grenzlinien sichtbar machen (im Stil als Ebene "parcel", Quelle "kataster", ab Zoom 16)
  map.setLayoutProperty('parcel', 'visibility', 'visible');
});
map.on('click', (e) => {
  const box = [[e.point.x - 6, e.point.y - 6], [e.point.x + 6, e.point.y + 6]];
  const f = map.queryRenderedFeatures(box, { layers: ['parcel'] })[0];
  const text = f ? (de ? `Grundstück ${f.properties.gstnr}, Katastralgemeinde ${f.properties.kgnr}` : `Parcel ${f.properties.gstnr}, cadastral municipality ${f.properties.kgnr}`) : (de ? 'Keine Grundstücksgrenze getroffen (sichtbar ab Zoom 16).' : 'No parcel boundary hit (visible from zoom 16).');
  new maplibregl.Popup().setLngLat(e.lngLat).setText(text).addTo(map);
});
````

## APIs used

- [Cadastre](https://docs.mapx.at/tilesets/cadastre.md) (vector tiles in the `satellite` style)

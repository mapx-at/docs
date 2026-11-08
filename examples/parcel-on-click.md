# Cadastre: parcel on click

Parcel boundaries over aerial imagery, a click shows the number.

The cadastre tileset contains boundary lines from zoom 16 with parcel number and cadastral municipality. `queryRenderedFeatures` reads them at the click point.

```js
// Kataster: Ein Klick auf eine Grundstücksgrenze zeigt Grundstücksnummer und Katastralgemeinde.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

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
  const text = f ? `Grundstück ${f.properties.gstnr}, KG ${f.properties.kgnr}` : 'Keine Grundstücksgrenze getroffen (ab Zoom 16 sichtbar).';
  new maplibregl.Popup().setLngLat(e.lngLat).setText(text).addTo(map);
});
```

## APIs used

- [Cadastre](/tilesets/cadastre/) (vector tiles in the `satellite` style)
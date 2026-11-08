# Kataster: Grundstück per Klick

Grundstücksgrenzen über dem Luftbild, ein Klick nennt die Nummer.

Das Kataster-Tileset enthält die Grenzlinien ab Zoom 16 mit Grundstücksnummer und Katastralgemeinde. `queryRenderedFeatures` liest sie am Klickpunkt.

```js
// Cadastre: a click on a parcel boundary shows the parcel number and cadastral municipality.
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
  const text = f ? `Parcel ${f.properties.gstnr}, cadastral municipality ${f.properties.kgnr}` : 'No parcel boundary hit (visible from zoom 16).';
  new maplibregl.Popup().setLngLat(e.lngLat).setText(text).addTo(map);
});
```

## Verwendete APIs

- [Kataster](/de/tilesets/cadastre/) (Vektorkacheln im Stil `satellite`)
# Switch the basemap

Switch between map, aerial imagery and terrain.

Three ready-made styles share position and zoom. `map.setStyle()` swaps the base, the view stays.

```js
// Basiskarte wechseln: Karte, Luftbild und Gelände sind drei fertige Stile.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STILE = {
  streets: 'https://api.mapx.at/v1/styles/streets',
  satellite: 'https://api.mapx.at/v1/styles/satellite',
  outdoors: 'https://api.mapx.at/v1/styles/outdoors',
};
const map = new maplibregl.Map({ container: 'map', style: STILE.streets, center: [11.4, 47.27], zoom: 11 });
map.addControl(new maplibregl.NavigationControl());

for (const knopf of document.querySelectorAll('[data-stil]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(STILE[knopf.dataset.stil]); // Position und Zoom bleiben erhalten
    for (const k of document.querySelectorAll('[data-stil]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
```

## APIs used

- [Styles](/api/styles/) `GET api.mapx.at/v1/styles/{streets|satellite|outdoors}`
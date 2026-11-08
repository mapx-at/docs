# Basiskarte wechseln


Drei fertige Stile teilen sich Position und Zoom. `map.setStyle()` tauscht die Basis, die Ansicht bleibt.

```js
// Switch the basemap: map, aerial imagery and terrain are three ready-made styles.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STILE = {
  streets: 'https://api.mapx.at/v1/styles/streets',
  satellite: 'https://api.mapx.at/v1/styles/satellite',
  outdoors: 'https://api.mapx.at/v1/styles/outdoors',
};
const map = new maplibregl.Map({ transformRequest, container: 'map', style: STILE.streets, center: [11.4, 47.27], zoom: 11 });
map.addControl(new maplibregl.NavigationControl());

for (const knopf of document.querySelectorAll('[data-stil]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(STILE[knopf.dataset.stil]); // position and zoom are kept
    for (const k of document.querySelectorAll('[data-stil]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
```

## Verwendete APIs

- [Styles](/de/api/styles/) `GET api.mapx.at/v1/styles/{streets|satellite|outdoors}`


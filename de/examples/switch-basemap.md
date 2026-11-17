# Basiskarte wechseln: Beispiel

Source: https://docs.mapx.at/de/examples/switch-basemap/

> Zwischen Karte, Luftbild (Orthophoto) und Gelände umschalten, mit MapLibre GL JS und mapx-Styles: lauffähiges Beispiel für einen Basiskarten-Umschalter.

Drei fertige Stile teilen sich Position und Zoom. `map.setStyle()` tauscht die Basis, die Ansicht bleibt.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

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
````

## Verwendete APIs

- [Styles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{streets|satellite|outdoors}`

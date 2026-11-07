# Basiskarte wechseln

Zwischen Karte, Luftbild und Gelände umschalten.

Drei fertige Stile teilen sich Position und Zoom. `map.setStyle()` tauscht die Basis, die Ansicht bleibt.

```js
// Basiskarte wechseln: Karte, Luftbild und Gelände sind drei fertige Stile.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STILE = {
  karte: 'https://mapx.at/styles/standard-tag-normal.json', // künftig /v1/styles/karte
  luftbild: 'https://mapx.at/styles/hybrid.json',           // künftig /v1/styles/luftbild
  gelaende: 'https://mapx.at/styles/gelaende.json',         // künftig /v1/styles/gelaende
};
const map = new maplibregl.Map({ container: 'map', style: STILE.karte, center: [11.4, 47.27], zoom: 11 });
map.addControl(new maplibregl.NavigationControl());

for (const knopf of document.querySelectorAll('[data-stil]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(STILE[knopf.dataset.stil]); // Position und Zoom bleiben erhalten
    for (const k of document.querySelectorAll('[data-stil]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
```

## Verwendete APIs

- Stile `karte`, `luftbild`, `gelaende`

> Stile aktuell von `https://mapx.at/styles/`; später `/v1/styles/{karte|luftbild|gelaende}`.
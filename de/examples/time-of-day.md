# Tageszeit und Nachtmodus

Den Standardstil in vier Lichtstimmungen.

Morgen, Tag, Abend und Nacht sind eigene Paletten desselben Stils. Für eine dunkle Oberfläche genügt die Nacht-Variante.

```js
// Tageszeit und Nachtmodus: die Standard-Stile gibt es für vier Lichtstimmungen.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STIL = { morgen: 'streets-dawn', tag: 'streets', abend: 'streets-dusk', nacht: 'streets-night' };
const stil = (licht) => `https://api.mapx.at/v1/styles/${STIL[licht]}`;
const map = new maplibregl.Map({ container: 'map', style: stil('tag'), center: [16.37, 48.21], zoom: 12 }); // Wien
map.addControl(new maplibregl.NavigationControl());

for (const knopf of document.querySelectorAll('[data-licht]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(stil(knopf.dataset.licht));
    for (const k of document.querySelectorAll('[data-licht]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
```

## Verwendete APIs

- [Styles](/de/api/styles/) `GET api.mapx.at/v1/styles/{streets-dawn|streets|streets-dusk|streets-night}`
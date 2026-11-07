# Time of day and night mode

The default style in four lighting moods.

Morning, day, evening and night are separate palettes of the same style. For a dark UI the night variant is enough.

```js
// Tageszeit und Nachtmodus: die Standard-Stile gibt es für vier Lichtstimmungen.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STIL = { morgen: 'karte-morgen', tag: 'karte', abend: 'karte-abend', nacht: 'karte-nacht' };
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

## APIs used

- [Styles](/en/api/styles/) `GET api.mapx.at/v1/styles/{karte-morgen|karte|karte-abend|karte-nacht}`
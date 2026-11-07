# Tageszeit und Nachtmodus

Den Standardstil in vier Lichtstimmungen.

Morgen, Tag, Abend und Nacht sind eigene Paletten desselben Stils. Für eine dunkle Oberfläche genügt die Nacht-Variante.

```js
// Tageszeit und Nachtmodus: die Standard-Stile gibt es für vier Lichtstimmungen.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const stil = (licht) => `https://mapx.at/styles/standard-${licht}-normal.json`; // licht: morgen, tag, abend, nacht
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

- Stil `karte` (Lichtvarianten)

> Lichtvarianten gibt es derzeit nur als Dateien unter `https://mapx.at/styles/standard-<licht>-normal.json`. Ob `/v1/styles/karte` sie als Parameter anbietet, steht noch nicht fest.
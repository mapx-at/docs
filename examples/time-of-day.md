# Time of day and night mode

The default style in four lighting moods.

Morning, day, evening and night are separate palettes of the same style. For a dark UI the night variant is enough.

```js
// Time of day and night mode: the standard styles come in four lighting moods.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const STIL = { morgen: 'streets-dawn', tag: 'streets', abend: 'streets-dusk', nacht: 'streets-night' };
const stil = (licht) => `https://api.mapx.at/v1/styles/${STIL[licht]}`;
const map = new maplibregl.Map({ transformRequest, container: 'map', style: stil('tag'), center: [16.37, 48.21], zoom: 12 }); // Wien
map.addControl(new maplibregl.NavigationControl());

for (const knopf of document.querySelectorAll('[data-licht]')) {
  knopf.addEventListener('click', () => {
    map.setStyle(stil(knopf.dataset.licht));
    for (const k of document.querySelectorAll('[data-licht]')) k.setAttribute('aria-pressed', String(k === knopf));
  });
}
```

## APIs used

- [Styles](/api/styles/) `GET api.mapx.at/v1/styles/{streets-dawn|streets|streets-dusk|streets-night}`
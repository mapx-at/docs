# Static Image fürs Exposé


Ein Static Image ist eine einfache URL, die ein PNG oder WebP liefert: für Exposés, E-Mails, PDFs und Orte ohne JavaScript. Stil, Marker, Ausschnitt und Größe stehen im Pfad. Karte anklicken verschiebt den Marker; das Bild darunter lädt die angezeigte URL.

```js
// Static image: click the map, get a ready-made PNG of that place for a listing, e-mail or PDF. No JavaScript map needed on the target page.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.3294, 48.2272], zoom: 14 });
map.addControl(new maplibregl.NavigationControl());
const img = document.getElementById('bild'), style = document.getElementById('style'), url = document.getElementById('url');
let at = [15.3294, 48.2272]; // Melk

function update() {
  const pin = `pin-l+e11d48(${at[0].toFixed(5)},${at[1].toFixed(5)})`;
  const u = `https://api.mapx.at/v1/static/${style.value}/${encodeURIComponent(pin)}/${at[0].toFixed(5)},${at[1].toFixed(5)},${Math.round(map.getZoom())}/480x300.webp`;
  img.src = u + '?access_token=' + TOKEN; url.textContent = u; // images cannot send headers, so the token goes in the query
}
map.on('click', (e) => { at = [e.lngLat.lng, e.lngLat.lat]; update(); });
style.addEventListener('change', update);
update();
```

## Verwendete APIs

- [Static Images](/de/api/static-images/) `GET api.mapx.at/v1/static/{style}/{overlay}/{view}/{size}`


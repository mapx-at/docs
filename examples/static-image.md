# Static image for a listing

Click a place, get a ready-made image URL with a pin.

A static image is a plain URL that returns a PNG or WebP: for listings, e-mails, PDFs and places without JavaScript. Style, pin, view and size are part of the path. Click the map to move the pin; the image below is the URL shown under it.

```js
// Static image: click the map, get a ready-made PNG of that place for a listing, e-mail or PDF. No JavaScript map needed on the target page.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.3294, 48.2272], zoom: 14 });
map.addControl(new maplibregl.NavigationControl());
const img = document.getElementById('bild'), style = document.getElementById('style'), url = document.getElementById('url');
let at = [15.3294, 48.2272]; // Melk

function update() {
  const pin = `pin-l+e11d48(${at[0].toFixed(5)},${at[1].toFixed(5)})`;
  const u = `https://api.mapx.at/v1/static/${style.value}/${encodeURIComponent(pin)}/${at[0].toFixed(5)},${at[1].toFixed(5)},${Math.round(map.getZoom())}/480x300.webp`;
  img.src = u; url.textContent = u;
}
map.on('click', (e) => { at = [e.lngLat.lng, e.lngLat.lat]; update(); });
style.addEventListener('change', update);
update();
```

## APIs used

- [Static Images](/api/static-images/) `GET api.mapx.at/v1/static/{style}/{overlay}/{view}/{size}`
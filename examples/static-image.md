# Static image example for a listing

Source: https://docs.mapx.at/examples/static-image/

> Click a place and get a ready-made static map image URL with a pin, for example for a real estate listing: static images API, runnable code.

A static image is a plain URL that returns a PNG or WebP: for listings, e-mails, PDFs and places without JavaScript. Style, pin, view and size are part of the path. Click the map to move the pin; the image below is the URL shown under it.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Static image: click the map, get a ready-made PNG of that place for a listing, e-mail or PDF. No JavaScript map needed on the target page.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.3294, 48.2272], zoom: 14 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
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
````

## APIs used

- [Static Images](https://docs.mapx.at/api/static-images.md) `GET api.mapx.at/v1/static/{style}/{overlay}/{view}/{size}`

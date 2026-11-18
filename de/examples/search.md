# Ortssuche: Beispiel mit Geocoding

Source: https://docs.mapx.at/de/examples/search/

> Orte, Adressen und Gipfel in Österreich suchen und auf der Karte anspringen: Beispiel zur Geocoding-API mit MapLibre GL JS und lauffähigem Code.

Die Geocoding-API liefert eine GeoJSON-`FeatureCollection` im Mapbox-Stil. Jeder Treffer hat `place_name`, `center` und meist eine `bbox` zum Zoomen.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Search: places, addresses, peaks and parcels via /v1/geocode (GeoJSON in the Mapbox shape).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [14.3, 47.6], zoom: 7 });
map.addControl(new maplibregl.NavigationControl());
let marker;

const eingabe = document.getElementById('q'), liste = document.getElementById('treffer');
eingabe.addEventListener('input', async () => {
  const q = eingabe.value.trim();
  liste.replaceChildren();
  if (q.length < 2) return;
  try {
    const r = await fetch(`https://api.mapx.at/v1/geocode?q=${encodeURIComponent(q)}&limit=6`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    for (const f of (await r.json()).features) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = f.place_name;
      b.addEventListener('click', () => {
        marker?.remove();
        marker = new maplibregl.Marker().setLngLat(f.center).addTo(map);
        f.bbox ? map.fitBounds([[f.bbox[0], f.bbox[1]], [f.bbox[2], f.bbox[3]]], { padding: 60, maxZoom: 17 }) : map.flyTo({ center: f.center, zoom: f.properties.zoom ?? 14 });
        liste.replaceChildren();
      });
      li.append(b); liste.append(li);
    }
  } catch (e) { liste.textContent = (de ? 'Fehler: ' : 'Error: ') + e.message; }
});
````

## Verwendete APIs

- [Geocoding](https://docs.mapx.at/de/api/geocoding.md) `GET api.mapx.at/v1/geocode`

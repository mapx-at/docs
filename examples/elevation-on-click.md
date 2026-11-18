# Elevation on click example

Source: https://docs.mapx.at/examples/elevation-on-click/

> A map click queries the elevation at a point, either terrain height or surface height including buildings, with the mapx elevation API.

The elevation API returns GeoJSON with `elevation` in metres. The DTM shows the terrain, the DSM buildings and vegetation.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Query elevation: a click asks /v1/elevation (DTM or DSM) for the point.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/outdoors', center: [12.694, 47.074], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
const popup = new maplibregl.Popup({ closeButton: false });

map.on('click', async (e) => {
  const modell = document.getElementById('modell').value; // dtm or dsm
  const url = `https://api.mapx.at/v1/elevation?points=${e.lngLat.lng.toFixed(6)},${e.lngLat.lat.toFixed(6)}&model=${modell}`;
  try {
    const { features } = await (await fetch(url, AUTH)).json();
    const h = features[0].properties.elevation;
    popup.setLngLat(e.lngLat).setText(h == null ? (de ? 'Außerhalb der Daten' : 'Outside the data') : `${h.toLocaleString(de ? 'de-AT' : 'en')} m (${modell.toUpperCase()})`).addTo(map);
  } catch (err) {
    popup.setLngLat(e.lngLat).setText((de ? 'Fehler: ' : 'Error: ') + err.message).addTo(map);
  }
});
````

## APIs used

- [Elevation](https://docs.mapx.at/api/elevation.md) `GET /v1/elevation`

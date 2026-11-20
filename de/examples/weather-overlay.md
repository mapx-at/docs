# Wetter-Overlay: Beispiel

Source: https://docs.mapx.at/de/examples/weather-overlay/

> Temperatur, Niederschlag oder Wind als Overlay über der Karte, Stunde per Regler: Wetterkacheln von GeoSphere Austria, lauffähiger Code.

Das TileJSON `/v1/weather/tiles/{parameter}.json` nennt die verfügbaren Stunden unter `mapx:time.times`. Jede Stunde ist eine eigene Kachelquelle, die Karte lädt sie als Raster-Layer. In OpenLayers ist jede Stunde eine eigene `ol.source.XYZ`.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Weather overlay: temperature (or precipitation, wind ...) as raster tiles over the map, hour selectable.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const API = 'https://api.mapx.at/v1';
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: API + '/styles/streets', center: [13.3, 47.6], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
let times = [];

async function laden() {
  const param = document.getElementById('param').value; // temperature, precipitation, wind ...
  const tj = await (await fetch(`${API}/weather/tiles/${param}.json`, AUTH)).json();
  times = tj['mapx:time'].times; // available hours (UTC)
  const slider = document.getElementById('stunde');
  slider.max = times.length - 1;
  zeigen(param, Number(slider.value));
}

function zeigen(param, i) {
  const t = times[Math.min(i, times.length - 1)];
  const source = document.documentElement.lang.startsWith('de') ? 'Quelle' : 'Source'; // page language
  document.getElementById('info').textContent = `${param}, ${t} (UTC), ${source}: GeoSphere Austria, CC BY 4.0`;
  if (map.getLayer('wetter')) map.removeLayer('wetter');
  if (map.getSource('wetter')) map.removeSource('wetter');
  map.addSource('wetter', {
    type: 'raster', tileSize: 512, maxzoom: 11, // smooth fields: fewer, larger tiles keep the request count low
    tiles: [`${API}/weather/tiles/${param}/${t}/{z}/{x}/{y}.png`],
    attribution: 'Wetter: GeoSphere Austria, CC BY 4.0',
  });
  map.addLayer({ id: 'wetter', type: 'raster', source: 'wetter', paint: { 'raster-opacity': 0.85, 'raster-fade-duration': 0 } });
}

map.on('load', laden);
document.getElementById('param').addEventListener('change', () => { document.getElementById('stunde').value = 0; laden(); });
document.getElementById('stunde').addEventListener('input', (e) => zeigen(document.getElementById('param').value, Number(e.target.value)));
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Weather overlay: temperature (or precipitation, wind ...) as raster tiles over the map, hour selectable.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const API = 'https://api.mapx.at/v1';
const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const wetter = new ol.layer.Tile({ opacity: 0.85 });
const map = new ol.Map({ target: 'map', layers: [base, wetter], view: new ol.View({ center: ol.proj.fromLonLat([13.3, 47.6]), zoom: 7.5 }) });
let times = [];

async function laden() {
  const param = document.getElementById('param').value; // temperature, precipitation, wind ...
  const tj = await (await fetch(`${API}/weather/tiles/${param}.json`, AUTH)).json();
  times = tj['mapx:time'].times; // available hours (UTC)
  const slider = document.getElementById('stunde');
  slider.max = times.length - 1;
  zeigen(param, Number(slider.value));
}

function zeigen(param, i) {
  const t = times[Math.min(i, times.length - 1)];
  const source = document.documentElement.lang.startsWith('de') ? 'Quelle' : 'Source'; // page language
  document.getElementById('info').textContent = `${param}, ${t} (UTC), ${source}: GeoSphere Austria, CC BY 4.0`;
  wetter.setSource(new ol.source.XYZ({
    url: `${API}/weather/tiles/${param}/${t}/{z}/{x}/{y}.png?access_token=` + TOKEN,
    tileSize: 512, maxZoom: 11, transition: 0, // smooth fields: fewer, larger tiles keep the request count low
    attributions: 'Wetter: GeoSphere Austria, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
  }));
}

laden();
document.getElementById('param').addEventListener('change', () => { document.getElementById('stunde').value = 0; laden(); });
document.getElementById('stunde').addEventListener('input', (e) => zeigen(document.getElementById('param').value, Number(e.target.value)));
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Weather overlay: temperature (or precipitation, wind ...) as raster tiles over the map, hour selectable.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style (CC BY 4.0), must stay visible on the map; the weather layer adds its own; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';
const API = 'https://api.mapx.at/v1';

const map = L.map('map').setView([47.6, 13.3], 7);
L.tileLayer(API + '/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
// smooth fields: 512 px tiles up to zoom 11 (Leaflet 12), Leaflet scales them up beyond
const wetter = L.tileLayer('', { tileSize: 512, zoomOffset: -1, maxNativeZoom: 12, maxZoom: 21, opacity: 0.85, attribution: 'Wetter: GeoSphere Austria, CC BY 4.0' }).addTo(map);
let times = [];
async function laden() {
  const param = document.getElementById('param').value; // temperature, precipitation, wind ...
  try {
    const res = await fetch(`${API}/weather/tiles/${param}.json`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    times = (await res.json())['mapx:time'].times; // available hours (UTC)
    const slider = document.getElementById('stunde');
    slider.max = times.length - 1;
    zeigen(param, Number(slider.value));
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
}
function zeigen(param, i) {
  const t = times[Math.min(i, times.length - 1)];
  document.getElementById('info').textContent = `${param}, ${t} (UTC), ${de ? 'Quelle' : 'Source'}: GeoSphere Austria, CC BY 4.0`;
  wetter.setUrl(`${API}/weather/tiles/${param}/${t}/{z}/{x}/{y}.png?access_token=${TOKEN}`);
}
laden();
document.getElementById('param').addEventListener('change', () => { document.getElementById('stunde').value = 0; laden(); });
document.getElementById('stunde').addEventListener('input', (e) => zeigen(document.getElementById('param').value, Number(e.target.value)));
````

> Datenquelle: GeoSphere Austria, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Die Namensnennung ist Pflicht.

In Leaflet bleibt es eine `L.tileLayer`, `setUrl()` wechselt Größe und Stunde.

## Verwendete APIs

- [Wetter](https://docs.mapx.at/de/api/weather.md#weather-raster-tile-png-256-transparent-outside-the-data) `GET api.mapx.at/v1/weather/tiles/{parameter}.json`, `GET api.mapx.at/v1/weather/tiles/{parameter}/{time}/{z}/{x}/{y}.png`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

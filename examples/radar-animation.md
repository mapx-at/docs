# Precipitation radar animation example

Source: https://docs.mapx.at/examples/radar-animation/

> Rain and snow of the next three hours in 15-minute steps, played as an animation on the map: weather nowcast of GeoSphere Austria, runnable code.

The TileJSON `/v1/weather/tiles/rr_nowcast.json` lists the 15-minute steps under `mapx:time.times`. The example adds one raster layer per step and switches their visibility, so the animation does not wait for tiles. For a single point use `GET /v1/weather/nowcast/{lon},{lat}`. OpenLayers loads a layer only once it becomes visible: the OpenLayers version fetches each step when first shown, later it comes from the cache.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Precipitation radar: nowcast tiles (15-minute steps, 3 hours) as an animation over the map.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const API = 'https://api.mapx.at/v1';
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: API + '/styles/streets', center: [13.3, 47.6], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)
let times = [], timer = null;

function zeigen(i) {
  const t = times[i];
  document.getElementById('zeit').value = i;
  const source = document.documentElement.lang.startsWith('de') ? 'Quelle' : 'Source'; // page language
  document.getElementById('info').textContent = `Radar nowcast ${t} (UTC), ${source}: GeoSphere Austria, CC BY 4.0`;
  // one layer per time step, only the selected one is visible: tiles are already loaded when playing
  times.forEach((_, k) => map.setLayoutProperty('radar-' + k, 'visibility', k === i ? 'visible' : 'none'));
}

function abspielen() {
  const btn = document.getElementById('play');
  if (timer) { clearInterval(timer); timer = null; btn.textContent = de ? 'Abspielen' : 'Play'; return; }
  btn.textContent = 'Pause';
  timer = setInterval(() => zeigen((Number(document.getElementById('zeit').value) + 1) % times.length), 700);
}

map.on('load', async () => {
  const tj = await (await fetch(`${API}/weather/tiles/rr_nowcast.json`, AUTH)).json();
  times = tj['mapx:time'].times; // 15-Minuten-Schritte (UTC)
  document.getElementById('zeit').max = times.length - 1;
  times.forEach((t, k) => {
    map.addSource('radar-' + k, {
      type: 'raster', tileSize: 512, maxzoom: 10,
      tiles: [tj['mapx:time'].template.replace('{time}', t)],
      attribution: 'Wetter: GeoSphere Austria, CC BY 4.0',
    });
    map.addLayer({ id: 'radar-' + k, type: 'raster', source: 'radar-' + k, layout: { visibility: 'none' }, paint: { 'raster-opacity': 0.85, 'raster-fade-duration': 0 } });
  });
  zeigen(0);
});
document.getElementById('play').addEventListener('click', abspielen);
document.getElementById('zeit').addEventListener('input', (e) => zeigen(Number(e.target.value)));
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Precipitation radar: nowcast tiles (15-minute steps, 3 hours) as an animation over the map.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const API = 'https://api.mapx.at/v1';
const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const map = new ol.Map({ target: 'map', layers: [base], view: new ol.View({ center: ol.proj.fromLonLat([13.3, 47.6]), zoom: 7.5 }) });
let times = [], layers = [], timer = null;

function zeigen(i) {
  const t = times[i];
  document.getElementById('zeit').value = i;
  const source = document.documentElement.lang.startsWith('de') ? 'Quelle' : 'Source'; // page language
  document.getElementById('info').textContent = `Radar nowcast ${t} (UTC), ${source}: GeoSphere Austria, CC BY 4.0`;
  // one layer per time step, only the selected one is visible: OpenLayers loads a step when first shown, later from the cache
  layers.forEach((l, k) => l.setVisible(k === i));
}

function abspielen() {
  const btn = document.getElementById('play');
  if (timer) { clearInterval(timer); timer = null; btn.textContent = de ? 'Abspielen' : 'Play'; return; }
  btn.textContent = 'Pause';
  timer = setInterval(() => zeigen((Number(document.getElementById('zeit').value) + 1) % times.length), 700);
}

(async () => {
  const tj = await (await fetch(`${API}/weather/tiles/rr_nowcast.json`, AUTH)).json();
  times = tj['mapx:time'].times; // 15-minute steps (UTC)
  document.getElementById('zeit').max = times.length - 1;
  layers = times.map((t, k) => new ol.layer.Tile({
    opacity: 0.85, visible: false,
    source: new ol.source.XYZ({ url: tj['mapx:time'].template.replace('{time}', t) + '?access_token=' + TOKEN, tileSize: 512, maxZoom: 10, transition: 0, attributions: k ? undefined : 'Wetter: GeoSphere Austria, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>' }),
  }));
  layers.forEach((l) => map.addLayer(l));
  zeigen(0);
})();
document.getElementById('play').addEventListener('click', abspielen);
document.getElementById('zeit').addEventListener('input', (e) => zeigen(Number(e.target.value)));
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Precipitation radar: nowcast tiles (15-minute steps, 3 hours) as an animation over the map.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style (CC BY 4.0), must stay visible on the map; the radar layers add their own; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';
const API = 'https://api.mapx.at/v1';

const map = L.map('map').setView([47.6, 13.3], 7);
L.tileLayer(API + '/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
let times = [], template = '', layers = [], timer = null;
function zeigen(i) {
  document.getElementById('zeit').value = i;
  document.getElementById('info').textContent = `Radar nowcast ${times[i]} (UTC), ${de ? 'Quelle' : 'Source'}: GeoSphere Austria, CC BY 4.0`;
  // one layer per time step, created on first use and then kept hidden: the second round plays without waiting for tiles
  layers[i] ??= L.tileLayer(template.replace('{time}', times[i]) + '?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, maxNativeZoom: 11, maxZoom: 21, opacity: 0, attribution: 'Wetter: GeoSphere Austria, CC BY 4.0' }).addTo(map);
  layers.forEach((l, k) => l?.setOpacity(k === i ? 0.85 : 0));
}
function abspielen() {
  const btn = document.getElementById('play');
  if (timer) { clearInterval(timer); timer = null; btn.textContent = de ? 'Abspielen' : 'Play'; return; }
  btn.textContent = 'Pause';
  timer = setInterval(() => zeigen((Number(document.getElementById('zeit').value) + 1) % times.length), 700);
}
(async () => {
  try {
    const res = await fetch(`${API}/weather/tiles/rr_nowcast.json`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const time = (await res.json())['mapx:time'];
    times = time.times; // 15-minute steps (UTC)
    template = time.template; // tile URL with {time}
    document.getElementById('zeit').max = times.length - 1;
    zeigen(0);
  } catch (err) { document.getElementById('info').textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
})();
document.getElementById('play').addEventListener('click', abspielen);
document.getElementById('zeit').addEventListener('input', (e) => zeigen(Number(e.target.value)));
````

> Data source: GeoSphere Austria, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution is required.

In Leaflet each step is its own `L.tileLayer`, switched via its opacity.

## APIs used

- [Weather](https://docs.mapx.at/api/weather.md#weather-raster-tile-png-256-transparent-outside-the-data) `GET api.mapx.at/v1/weather/tiles/rr_nowcast.json`, `GET api.mapx.at/v1/weather/tiles/rr_nowcast/{time}/{z}/{x}/{y}.png`
- [Precipitation nowcast](https://docs.mapx.at/api/weather.md) `GET api.mapx.at/v1/weather/nowcast/{lon},{lat}`
- [Static Tiles](https://docs.mapx.at/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

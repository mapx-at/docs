# Wetter-Overlay

Temperatur, Niederschlag oder Wind als Overlay über der Karte, Stunde per Regler.

Das TileJSON `/v1/weather/tiles/{parameter}.json` nennt die verfügbaren Stunden unter `mapx:time.times`. Jede Stunde ist eine eigene Kachelquelle, die Karte lädt sie als Raster-Layer.

```js
// Wetter-Overlay: Temperatur (oder Niederschlag, Wind ...) als Rasterkacheln über der Karte, Stunde wählbar.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const API = 'https://api.mapx.at/v1';
const map = new maplibregl.Map({ transformRequest, container: 'map', style: API + '/styles/streets', center: [13.3, 47.6], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
let times = [];

async function laden() {
  const param = document.getElementById('param').value; // temperature, precipitation, wind ...
  const tj = await (await fetch(`${API}/weather/tiles/${param}.json`, AUTH)).json();
  times = tj['mapx:time'].times; // verfügbare Stunden (UTC)
  const slider = document.getElementById('stunde');
  slider.max = times.length - 1;
  zeigen(param, Number(slider.value));
}

function zeigen(param, i) {
  const t = times[Math.min(i, times.length - 1)];
  document.getElementById('info').textContent = `${param}, ${t} (UTC), Quelle: GeoSphere Austria, CC BY 4.0`;
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
```

> Datenquelle: GeoSphere Austria, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Die Namensnennung ist Pflicht.

## Verwendete APIs

- [Wetter](/de/api/weather/#weather-raster-tile-png-256-transparent-outside-the-data) `GET api.mapx.at/v1/weather/tiles/{parameter}.json`, `GET api.mapx.at/v1/weather/tiles/{parameter}/{time}/{z}/{x}/{y}.png`
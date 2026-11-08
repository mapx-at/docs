# Precipitation radar animation

Rain and snow of the next three hours in 15-minute steps, played as an animation.

The TileJSON `/v1/weather/tiles/rr_nowcast.json` lists the 15-minute steps under `mapx:time.times`. The example adds one raster layer per step and switches their visibility, so the animation does not wait for tiles. For a single point use `GET /v1/weather/nowcast/{lon},{lat}`.

```js
// Precipitation radar: nowcast tiles (15-minute steps, 3 hours) as an animation over the map.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const API = 'https://api.mapx.at/v1';
const map = new maplibregl.Map({ transformRequest, container: 'map', style: API + '/styles/streets', center: [13.3, 47.6], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
let times = [], timer = null;

function zeigen(i) {
  const t = times[i];
  document.getElementById('zeit').value = i;
  document.getElementById('info').textContent = `Radar-Nowcast ${t} (UTC), Quelle: GeoSphere Austria, CC BY 4.0`;
  // one layer per time step, only the selected one is visible: tiles are already loaded when playing
  times.forEach((_, k) => map.setLayoutProperty('radar-' + k, 'visibility', k === i ? 'visible' : 'none'));
}

function abspielen() {
  const btn = document.getElementById('play');
  if (timer) { clearInterval(timer); timer = null; btn.textContent = 'Play'; return; }
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
```

> Data source: GeoSphere Austria, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Attribution is required.

## APIs used

- [Weather](/api/weather/#weather-raster-tile-png-256-transparent-outside-the-data) `GET api.mapx.at/v1/weather/tiles/rr_nowcast.json`, `GET api.mapx.at/v1/weather/tiles/rr_nowcast/{time}/{z}/{x}/{y}.png`
- [Precipitation nowcast](/api/weather/) `GET api.mapx.at/v1/weather/nowcast/{lon},{lat}`
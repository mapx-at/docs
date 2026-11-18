# Departures at a stop example

Source: https://docs.mapx.at/examples/departures/

> Click near a public transport stop in Austria and see the next departures: transit departures API with a live map and runnable code.

A click finds the nearest stop within 150 m via `/v1/stops` and lists, via `/v1/departures`, the next departures from the scheduled timetable (no real-time data). The dots are the stops of the visible area from `/v1/stops`.

## Code

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";

// Departures: click near a stop to see the next departures (/v1/stops finds the stop, /v1/departures lists the departures).
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
const api = async (path) => { const r = await fetch('https://api.mapx.at/v1' + path, AUTH); if (!r.ok) throw new Error((await r.json()).title || r.statusText); return r.json(); };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [16.3725, 48.2085], zoom: 15.5 });
map.addControl(new maplibregl.NavigationControl());
const popup = new maplibregl.Popup({ maxWidth: '340px' });

// Show the stops of the visible area as dots.
map.on('load', () => {
  map.addSource('stops', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'stops', type: 'circle', source: 'stops', paint: { 'circle-radius': 6, 'circle-color': '#2346d8', 'circle-stroke-color': '#fff', 'circle-stroke-width': 2 } });
  const load = async () => {
    const b = map.getBounds();
    const { stops } = await api(`/stops?bbox=${[b.getWest(), b.getSouth(), b.getEast(), b.getNorth()].map((v) => v.toFixed(5)).join(',')}&limit=100`);
    map.getSource('stops').setData({ type: 'FeatureCollection', features: stops.map((s) => ({ type: 'Feature', properties: { name: s.name }, geometry: { type: 'Point', coordinates: s.location } })) });
  };
  map.on('moveend', load);
  load();
});

// A click finds the nearest stop within 150 m and lists its next 8 departures (timetable, no real-time data).
let seq = 0;
map.on('click', async (e) => {
  const n = ++seq, at = `${e.lngLat.lng.toFixed(6)},${e.lngLat.lat.toFixed(6)}`;
  popup.setLngLat(e.lngLat).setText(de ? 'Lade …' : 'Loading …').addTo(map);
  try {
    const stop = (await api(`/stops?near=${at}&radius=150&limit=1`)).stops[0];
    if (!stop) { popup.setText(de ? 'Keine Haltestelle in 150 m' : 'No stop within 150 m'); return; }
    const { departures } = await api(`/departures?stop=${encodeURIComponent(stop.id)}&limit=8`);
    if (n !== seq) return; // a newer click has taken over
    const el = document.createElement('div');
    el.append(Object.assign(document.createElement('strong'), { textContent: stop.name }));
    if (!departures.length) el.append(Object.assign(document.createElement('div'), { textContent: de ? 'Keine Abfahrten' : 'No departures' }));
    for (const d of departures) {
      const row = document.createElement('div');
      row.append(Object.assign(document.createElement('b'), { textContent: d.time.slice(11, 16) }), ` ${d.line.name} → ${d.headsign}`);
      el.append(row);
    }
    popup.setLngLat(stop.location).setDOMContent(el);
  } catch (err) { if (n === seq) popup.setText(err.message); }
});
````

## APIs used

- [Transit](https://docs.mapx.at/api/transit.md) `GET api.mapx.at/v1/stops`
- [Transit](https://docs.mapx.at/api/transit.md) `GET api.mapx.at/v1/departures`

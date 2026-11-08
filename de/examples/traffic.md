# Verkehrslage und Meldungen

Verkehrsstil plus die aktiven Meldungen im sichtbaren Ausschnitt.

Der Stil `traffic-day` färbt das Straßennetz nach der aktuellen Verkehrslage. Meldungen (Unfälle, Baustellen, Sperren) kommen von `/v1/traffic/incidents`; das Beispiel lädt die aktiven für den sichtbaren Ausschnitt und zeichnet sie darüber. Eine Linie anklicken zeigt den Text.

```js
// Traffic: live traffic flow as a style plus the current incident list for the visible area.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/traffic-day', center: [16.2, 48.1], zoom: 9 });
map.addControl(new maplibregl.NavigationControl());
const list = document.getElementById('meldungen');
const popup = new maplibregl.Popup();

map.on('load', () => {
  map.addSource('incidents', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'incidents', type: 'line', source: 'incidents', layout: { 'line-cap': 'round' }, paint: { 'line-color': ['case', ['get', 'closed'], '#c8372d', '#e8890c'], 'line-width': 6, 'line-opacity': 0.8 } });
  map.on('click', 'incidents', (e) => popup.setLngLat(e.lngLat).setText(e.features[0].properties.text).addTo(map));
  const load = async () => {
    const b = map.getBounds();
    const res = await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${b.getWest()},${b.getSouth()},${b.getEast()},${b.getNorth()}&active=true&limit=50`, AUTH);
    const fc = await res.json();
    map.getSource('incidents').setData(fc);
    list.textContent = `${fc.features.length} active incident(s) in view` + (fc.features[0] ? `, e.g. ${fc.features[0].properties.road ?? ''} ${fc.features[0].properties.type}` : '');
  };
  map.on('moveend', load);
  load();
});
```

## Verwendete APIs

- [Verkehr](/de/api/traffic/) `GET api.mapx.at/v1/traffic/incidents`
- [Styles](/de/api/styles/) `GET api.mapx.at/v1/styles/traffic-day`
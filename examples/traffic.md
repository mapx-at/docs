# Traffic flow and incidents

Traffic style plus the active incidents of the visible area.

The style `traffic-day` colours the road network by current flow. Incidents (accidents, roadworks, closures) come from `/v1/traffic/incidents`; the example loads the active ones for the visible area and draws them on top. Click a line for the text.

```js
// Traffic: live traffic flow as a style plus the current incident list for the visible area.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/traffic-day', center: [16.2, 48.1], zoom: 9 });
map.addControl(new maplibregl.NavigationControl());
const list = document.getElementById('meldungen');
const popup = new maplibregl.Popup();

map.on('load', () => {
  map.addSource('incidents', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'incidents', type: 'line', source: 'incidents', layout: { 'line-cap': 'round' }, paint: { 'line-color': ['case', ['get', 'closed'], '#c8372d', '#e8890c'], 'line-width': 6, 'line-opacity': 0.8 } });
  map.on('click', 'incidents', (e) => popup.setLngLat(e.lngLat).setText(e.features[0].properties.text).addTo(map));
  const load = async () => {
    const b = map.getBounds();
    const res = await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${b.getWest()},${b.getSouth()},${b.getEast()},${b.getNorth()}&active=true&limit=50`);
    const fc = await res.json();
    map.getSource('incidents').setData(fc);
    list.textContent = `${fc.features.length} active incident(s) in view` + (fc.features[0] ? `, e.g. ${fc.features[0].properties.road ?? ''} ${fc.features[0].properties.type}` : '');
  };
  map.on('moveend', load);
  load();
});
```

## APIs used

- [Traffic](/api/traffic/) `GET api.mapx.at/v1/traffic/incidents`
- [Styles](/api/styles/) `GET api.mapx.at/v1/styles/traffic-day`
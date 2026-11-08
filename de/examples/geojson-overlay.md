# Eigenes GeoJSON auf der Karte

Punkte, Linie und Fläche über dem mapx-Stil, Popup per Klick.

Eigene Daten liegen als GeoJSON-Quelle über dem mapx-Stil. Ein Filter auf den Geometrietyp erlaubt, Punkte, Linien und Flächen getrennt zu gestalten. Größere Datensätze funktionieren genauso als PMTiles.

```js
// Own data: add a GeoJSON layer (points, line, area) on top of the mapx style and show a popup on click.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const data = {
  type: 'FeatureCollection',
  features: [
    { type: 'Feature', properties: { name: 'Melk Abbey' }, geometry: { type: 'Point', coordinates: [15.3333, 48.2279] } },
    { type: 'Feature', properties: { name: 'Krems' }, geometry: { type: 'Point', coordinates: [15.6141, 48.4097] } },
    { type: 'Feature', properties: { name: 'Danube cycle path (section)' }, geometry: { type: 'LineString', coordinates: [[15.3333, 48.2279], [15.4, 48.3], [15.5, 48.38], [15.6141, 48.4097]] } },
    { type: 'Feature', properties: { name: 'Wachau (rough area)' }, geometry: { type: 'Polygon', coordinates: [[[15.3, 48.2], [15.7, 48.35], [15.7, 48.45], [15.4, 48.4], [15.3, 48.2]]] } },
  ],
};
const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.45, 48.3], zoom: 9.5 });
map.addControl(new maplibregl.NavigationControl());

map.on('load', () => {
  map.addSource('own', { type: 'geojson', data });
  map.addLayer({ id: 'own-area', type: 'fill', source: 'own', filter: ['==', '$type', 'Polygon'], paint: { 'fill-color': '#2346d8', 'fill-opacity': 0.12 } });
  map.addLayer({ id: 'own-line', type: 'line', source: 'own', filter: ['==', '$type', 'LineString'], paint: { 'line-color': '#2346d8', 'line-width': 4 } });
  map.addLayer({ id: 'own-points', type: 'circle', source: 'own', filter: ['==', '$type', 'Point'], paint: { 'circle-radius': 8, 'circle-color': '#e11d48', 'circle-stroke-color': '#fff', 'circle-stroke-width': 2 } });
  map.on('click', 'own-points', (e) => new maplibregl.Popup().setLngLat(e.lngLat).setText(e.features[0].properties.name).addTo(map));
  map.on('mouseenter', 'own-points', () => (map.getCanvas().style.cursor = 'pointer'));
  map.on('mouseleave', 'own-points', () => (map.getCanvas().style.cursor = ''));
  // PMTiles of your own work the same way: map.addSource('x', { type: 'vector', url: 'pmtiles://https://example.com/x.pmtiles' })
});
```

## Verwendete APIs

- [Styles](/de/api/styles/) `GET api.mapx.at/v1/styles/{id}`
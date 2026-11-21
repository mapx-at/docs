# Eigenes GeoJSON auf der Karte: Beispiel

Source: https://docs.mapx.at/de/examples/geojson-overlay/

> Eigene GeoJSON-Punkte, -Linie und -Fläche über dem mapx-Stil zeigen, mit Popup per Klick, in MapLibre GL JS. Lauffähiger Code zum Kopieren.

Eigene Daten liegen als GeoJSON-Quelle über dem mapx-Stil. Ein Filter auf den Geometrietyp erlaubt, Punkte, Linien und Flächen getrennt zu gestalten. Größere Datensätze funktionieren genauso als PMTiles. In OpenLayers wählt eine Stilfunktion den Stil je Geometrietyp.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Own data: add a GeoJSON layer (points, line, area) on top of the mapx style and show a popup on click.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const data = {
  type: 'FeatureCollection',
  features: [
    { type: 'Feature', properties: { name: de ? 'Stift Melk' : 'Melk Abbey' }, geometry: { type: 'Point', coordinates: [15.3333, 48.2279] } },
    { type: 'Feature', properties: { name: 'Krems' }, geometry: { type: 'Point', coordinates: [15.6141, 48.4097] } },
    { type: 'Feature', properties: { name: de ? 'Donauradweg (Abschnitt)' : 'Danube cycle path (section)' }, geometry: { type: 'LineString', coordinates: [[15.3333, 48.2279], [15.4, 48.3], [15.5, 48.38], [15.6141, 48.4097]] } },
    { type: 'Feature', properties: { name: de ? 'Wachau (grobe Fläche)' : 'Wachau (rough area)' }, geometry: { type: 'Polygon', coordinates: [[[15.3, 48.2], [15.7, 48.35], [15.7, 48.45], [15.4, 48.4], [15.3, 48.2]]] } },
  ],
};
const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.45, 48.3], zoom: 9.5 });
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

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
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Own data: add a GeoJSON layer (points, line, area) on top of the mapx map and show a popup on click.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const data = {
  type: 'FeatureCollection',
  features: [
    { type: 'Feature', properties: { name: de ? 'Stift Melk' : 'Melk Abbey' }, geometry: { type: 'Point', coordinates: [15.3333, 48.2279] } },
    { type: 'Feature', properties: { name: 'Krems' }, geometry: { type: 'Point', coordinates: [15.6141, 48.4097] } },
    { type: 'Feature', properties: { name: de ? 'Donauradweg (Abschnitt)' : 'Danube cycle path (section)' }, geometry: { type: 'LineString', coordinates: [[15.3333, 48.2279], [15.4, 48.3], [15.5, 48.38], [15.6141, 48.4097]] } },
    { type: 'Feature', properties: { name: de ? 'Wachau (grobe Fläche)' : 'Wachau (rough area)' }, geometry: { type: 'Polygon', coordinates: [[[15.3, 48.2], [15.7, 48.35], [15.7, 48.45], [15.4, 48.4], [15.3, 48.2]]] } },
  ],
};
const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const STYLE = {
  Polygon: new ol.style.Style({ fill: new ol.style.Fill({ color: 'rgba(35, 70, 216, 0.12)' }) }),
  LineString: new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#2346d8', width: 4 }) }),
  Point: new ol.style.Style({ zIndex: 1, image: new ol.style.Circle({ radius: 8, fill: new ol.style.Fill({ color: '#e11d48' }), stroke: new ol.style.Stroke({ color: '#fff', width: 2 }) }) }),
};
const own = new ol.source.Vector({ features: new ol.format.GeoJSON({ featureProjection: 'EPSG:3857' }).readFeatures(data) });
const label = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
label.style.cssText = 'background:#fff;color:#000;padding:4px 8px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006';
const popup = new ol.Overlay({ element: label, positioning: 'bottom-center', offset: [0, -8] });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: own, style: (f) => STYLE[f.getGeometry().getType()] })], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([15.45, 48.3]), zoom: 10.5 }) });

const pointAt = (pixel) => map.forEachFeatureAtPixel(pixel, (f) => (f.getGeometry().getType() === 'Point' ? f : undefined));
map.on('click', (e) => {
  const f = pointAt(e.pixel);
  label.textContent = f?.get('name') ?? '';
  popup.setPosition(f?.getGeometry().getCoordinates());
});
map.on('pointermove', (e) => (map.getTargetElement().style.cursor = pointAt(e.pixel) ? 'pointer' : ''));
// Your own vector tiles work the same way: new ol.layer.VectorTile({ source: new ol.source.VectorTile({ format: new ol.format.MVT(), url: 'https://example.com/{z}/{x}/{y}.pbf' }) })
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Own data: add a GeoJSON layer (points, line, area) on top of the mapx raster tiles and show a popup on click.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources of the style (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const data = {
  type: 'FeatureCollection',
  features: [
    { type: 'Feature', properties: { name: de ? 'Stift Melk' : 'Melk Abbey' }, geometry: { type: 'Point', coordinates: [15.3333, 48.2279] } },
    { type: 'Feature', properties: { name: 'Krems' }, geometry: { type: 'Point', coordinates: [15.6141, 48.4097] } },
    { type: 'Feature', properties: { name: de ? 'Donauradweg (Abschnitt)' : 'Danube cycle path (section)' }, geometry: { type: 'LineString', coordinates: [[15.3333, 48.2279], [15.4, 48.3], [15.5, 48.38], [15.6141, 48.4097]] } },
    { type: 'Feature', properties: { name: de ? 'Wachau (grobe Fläche)' : 'Wachau (rough area)' }, geometry: { type: 'Polygon', coordinates: [[[15.3, 48.2], [15.7, 48.35], [15.7, 48.45], [15.4, 48.4], [15.3, 48.2]]] } },
  ],
};
const map = L.map('map').setView([48.3, 15.45], 10);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
L.geoJSON(data, {
  interactive: false, // area and line let clicks through to the points (circles set their own options)
  style: (f) => ({ Polygon: { stroke: false, fillColor: '#2346d8', fillOpacity: 0.12 }, LineString: { color: '#2346d8', weight: 4, opacity: 1 } })[f.geometry.type], // one style per geometry type
  pointToLayer: (f, latlng) => L.circleMarker(latlng, { radius: 8, color: '#fff', weight: 2, fillColor: '#e11d48', fillOpacity: 1 }),
  onEachFeature: (f, layer) => f.geometry.type === 'Point' && layer.bindPopup(Object.assign(document.createElement('span'), { textContent: f.properties.name })), // as text, not HTML
}).addTo(map);
````

In Leaflet übernimmt `L.geoJSON` mit einer `style`-Funktion je Geometrietyp diese Aufgabe, Punkte werden mit `pointToLayer` zu Kreisen.

## Verwendete APIs

- [Styles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}`
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

# Kataster: Grundstück per Klick

Source: https://docs.mapx.at/de/examples/parcel-on-click/

> Grundstücksgrenzen (Kataster) über dem Luftbild zeigen, ein Klick nennt die Grundstücksnummer: Kataster-Tileset und Parcel-API, lauffähiger Code.

Das Kataster-Tileset enthält die Grenzlinien ab Zoom 16 mit Grundstücksnummer und Katastralgemeinde. `queryRenderedFeatures` liest sie am Klickpunkt. OpenLayers und Leaflet zeigen das Luftbild als Rasterkacheln ohne Grenzlinien; dort fragt ein Klick die Parcel-API und zeichnet den Umriss des Grundstücks.

## Code

### MapLibre GL JS

````js
import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";

// Cadastre: a click on a parcel boundary shows the parcel number and cadastral municipality.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ attributionControl: false, transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/satellite', center: [15.3306, 48.2275], zoom: 18 }); // Melk
map.addControl(new maplibregl.NavigationControl());
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once('load', () => map.getContainer().querySelector('.maplibregl-compact-show')?.classList.remove('maplibregl-compact-show')); // start collapsed (MapLibre keeps it open until the first drag)

map.on('load', () => {
  // Grenzlinien sichtbar machen (im Stil als Ebene "parcel", Quelle "cadastre", ab Zoom 16)
  map.setLayoutProperty('parcel', 'visibility', 'visible');
});
map.on('click', (e) => {
  const box = [[e.point.x - 6, e.point.y - 6], [e.point.x + 6, e.point.y + 6]];
  const f = map.queryRenderedFeatures(box, { layers: ['parcel'] })[0];
  const text = f ? (de ? `Grundstück ${f.properties.gstnr}, Katastralgemeinde ${f.properties.kgnr}` : `Parcel ${f.properties.gstnr}, cadastral municipality ${f.properties.kgnr}`) : (de ? 'Keine Grundstücksgrenze getroffen (sichtbar ab Zoom 16).' : 'No parcel boundary hit (visible from zoom 16).');
  new maplibregl.Popup().setLngLat(e.lngLat).setText(text).addTo(map);
});
````

### OpenLayers

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
```

````js
// Cadastre: a click asks the Parcel API for the parcel at that point, draws its outline and shows parcel number and cadastral municipality.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = 'Orthophoto: <a href="https://www.bev.gv.at">BEV</a>, <a href="https://basemap.at">basemap.at</a>; BEV – DKM; <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const base = new ol.layer.Tile({ source: new ol.source.XYZ({ url: 'https://api.mapx.at/v1/styles/satellite/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, tileSize: 512, maxZoom: 20, attributions: ATTRIBUTION, attributionsCollapsible: false }) });
const parcel = new ol.source.Vector();
const outline = new ol.style.Style({ stroke: new ol.style.Stroke({ color: '#ffd400', width: 3 }), fill: new ol.style.Fill({ color: 'rgba(255, 212, 0, 0.15)' }) });
const label = document.createElement('div'); // OpenLayers has no popup of its own: an overlay with a plain element
label.style.cssText = 'background:#fff;color:#000;padding:4px 8px;border-radius:6px;font:13px system-ui,sans-serif;box-shadow:0 1px 4px #0006';
const popup = new ol.Overlay({ element: label, positioning: 'bottom-center', offset: [0, -8] });
const map = new ol.Map({ target: 'map', layers: [base, new ol.layer.Vector({ source: parcel, style: outline })], overlays: [popup], view: new ol.View({ center: ol.proj.fromLonLat([15.3306, 48.2275]), zoom: 19, maxZoom: 21 }) }); // Melk

map.on('click', async (e) => {
  const [lon, lat] = ol.proj.toLonLat(e.coordinate);
  parcel.clear();
  try {
    const r = await fetch(`https://api.mapx.at/v1/parcel/${lon.toFixed(6)},${lat.toFixed(6)}`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const f = (await r.json()).features[0]; // GeoJSON polygon of the parcel
    if (f) parcel.addFeature(new ol.format.GeoJSON().readFeature(f, { featureProjection: 'EPSG:3857' }));
    const p = f?.properties;
    label.textContent = p ? (de ? `Grundstück ${p.parcel_number}, Katastralgemeinde ${p.cadastral_municipality_id}` : `Parcel ${p.parcel_number}, cadastral municipality ${p.cadastral_municipality_id}`) : (de ? 'Kein Grundstück an dieser Stelle.' : 'No parcel at this point.');
  } catch (err) { label.textContent = (de ? 'Fehler: ' : 'Error: ') + err.message; }
  popup.setPosition(e.coordinate);
});
````

### Leaflet

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

````js
// Cadastre: a click asks the Parcel API for the parcel at that point, draws its outline and shows parcel number and cadastral municipality.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };
const de = document.documentElement.lang.startsWith('de'); // page language: German or English texts
// Data sources (CC BY 4.0), must stay visible on the map; full text per source: https://docs.mapx.at/reference/attribution/
const ATTRIBUTION = 'Orthophoto: <a href="https://www.bev.gv.at">BEV</a>, <a href="https://basemap.at">basemap.at</a>; BEV – DKM; <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>';

const map = L.map('map').setView([48.2275, 15.3306], 19); // Melk
L.tileLayer('https://api.mapx.at/v1/styles/satellite/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, { tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21, attribution: ATTRIBUTION }).addTo(map);
const parcel = L.geoJSON(null, { style: { color: '#ffd400', weight: 3, fillOpacity: 0.15 } }).addTo(map);

map.on('click', async (e) => {
  const { lng, lat } = e.latlng;
  parcel.clearLayers();
  let text;
  try {
    const r = await fetch(`https://api.mapx.at/v1/parcel/${lng.toFixed(6)},${lat.toFixed(6)}`, AUTH);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const f = (await r.json()).features[0]; // GeoJSON polygon of the parcel
    if (f) parcel.addData(f);
    const p = f?.properties;
    text = p ? (de ? `Grundstück ${p.parcel_number}, Katastralgemeinde ${p.cadastral_municipality_id}` : `Parcel ${p.parcel_number}, cadastral municipality ${p.cadastral_municipality_id}`) : (de ? 'Kein Grundstück an dieser Stelle.' : 'No parcel at this point.');
  } catch (err) { text = (de ? 'Fehler: ' : 'Error: ') + err.message; }
  L.popup().setLatLng(e.latlng).setContent(Object.assign(document.createElement('span'), { textContent: text })).openOn(map); // as text, not HTML
});
````

## Verwendete APIs

- [Kataster](https://docs.mapx.at/de/tilesets/cadastre.md) (Vektorkacheln im Stil `satellite`)
- [Parcel](https://docs.mapx.at/de/api/tilequery.md) `GET api.mapx.at/v1/parcel/{lon},{lat}` (OpenLayers, Leaflet)
- [Static Tiles](https://docs.mapx.at/de/api/styles.md) `GET api.mapx.at/v1/styles/{id}/tiles/512/{z}/{x}/{y}` (OpenLayers, Leaflet)

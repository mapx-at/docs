# Erreichbarkeit (Isochronen)

Welche Fläche ist in 5, 10 und 15 Minuten erreichbar?

Ein Klick setzt den Startpunkt. Die Isochronen-API liefert je Stufe ein MultiPolygon, die größte Fläche zuerst.

> Such- und Routing-Dienst antworten nur auf freigegebene Herkünfte (CORS). Die Beispiele laufen hier, sobald `docs.mapx.at` freigegeben ist; auf eigenen Seiten braucht es ebenfalls eine Freigabe der Herkunft.

```js
// Isochronen: Flächen, die man in 5, 10 und 15 Minuten erreicht.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://mapx.at/styles/standard-tag-normal.json', center: [15.33, 48.23], zoom: 13 });
map.addControl(new maplibregl.NavigationControl());

map.on('load', () => {
  map.addSource('iso', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'iso-flaeche', type: 'fill', source: 'iso', paint: { 'fill-color': '#2346d8', 'fill-opacity': 0.12 } });
  map.addLayer({ id: 'iso-linie', type: 'line', source: 'iso', paint: { 'line-color': '#2346d8', 'line-width': 2 } });
  abfragen([15.33, 48.23]);
});
map.on('click', (e) => abfragen([e.lngLat.lng, e.lngLat.lat]));

async function abfragen(ort) {
  const profil = document.getElementById('profil').value; // foot, bike oder car
  const url = `https://routing.mapx.at/isochrone?profile=${profil}&location=${ort.join(',')}&contours=5,10,15`; // künftig /v1/isochrone
  try {
    const r = await fetch(url);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    map.getSource('iso').setData(await r.json()); // MultiPolygon je Stufe, größte zuerst
  } catch (err) { document.getElementById('info').textContent = 'Fehler: ' + err.message; }
}
```

## Verwendete APIs

- Routing `GET routing.mapx.at/isochrone`

> Aktuell `routing.mapx.at/isochrone`; `GET /v1/isochrone` ist geplant.
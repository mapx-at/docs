# Elevation on click

A click queries the elevation at a point, terrain or surface.

The elevation API returns GeoJSON with `elevation` in metres. The DTM shows the terrain, the DSM buildings and vegetation.

```js
// Höhe abfragen: ein Klick fragt /v1/elevation (DTM oder DSM) für den Punkt.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/outdoors', center: [12.694, 47.074], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
const popup = new maplibregl.Popup({ closeButton: false });

map.on('click', async (e) => {
  const modell = document.getElementById('modell').value; // dtm oder dsm
  const url = `https://api.mapx.at/v1/elevation?points=${e.lngLat.lng.toFixed(6)},${e.lngLat.lat.toFixed(6)}&model=${modell}`;
  try {
    const { features } = await (await fetch(url)).json();
    const h = features[0].properties.elevation;
    popup.setLngLat(e.lngLat).setText(h == null ? 'Außerhalb der Daten' : `${h.toLocaleString('de-AT')} m (${modell.toUpperCase()})`).addTo(map);
  } catch (err) {
    popup.setLngLat(e.lngLat).setText('Fehler: ' + err.message).addTo(map);
  }
});
```

## APIs used

- [Elevation](/en/api/hoehe/) `GET /v1/elevation`
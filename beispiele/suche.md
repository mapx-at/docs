# Suche

Orte, Adressen und Gipfel suchen und anspringen.

Die Geocoding-API liefert eine GeoJSON-`FeatureCollection` im Mapbox-Stil. Jeder Treffer hat `place_name`, `center` und meist eine `bbox` zum Zoomen. Kein Zugangstoken nötig.

```js
// Suche: Orte, Adressen, Gipfel und Grundstücke über /v1/geocode (GeoJSON im Mapbox-Stil).
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [14.3, 47.6], zoom: 7 });
map.addControl(new maplibregl.NavigationControl());
let marker;

const eingabe = document.getElementById('q'), liste = document.getElementById('treffer');
eingabe.addEventListener('input', async () => {
  const q = eingabe.value.trim();
  liste.replaceChildren();
  if (q.length < 2) return;
  try {
    const r = await fetch(`https://api.mapx.at/v1/geocode?q=${encodeURIComponent(q)}&limit=6`);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    for (const f of (await r.json()).features) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = f.place_name;
      b.addEventListener('click', () => {
        marker?.remove();
        marker = new maplibregl.Marker().setLngLat(f.center).addTo(map);
        f.bbox ? map.fitBounds([[f.bbox[0], f.bbox[1]], [f.bbox[2], f.bbox[3]]], { padding: 60, maxZoom: 17 }) : map.flyTo({ center: f.center, zoom: f.properties.zoom ?? 14 });
        liste.replaceChildren();
      });
      li.append(b); liste.append(li);
    }
  } catch (e) { liste.textContent = 'Fehler: ' + e.message; }
});
```

## Verwendete APIs

- [Geocoding](/api/geocoding/) `GET api.mapx.at/v1/geocode`
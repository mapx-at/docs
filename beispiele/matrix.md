# Fahrzeitmatrix

Fahrzeiten zwischen mehreren Orten in einer Anfrage.

Die Matrix-API nimmt Quellen (und optional Ziele) und liefert Dauer (und optional Distanz) für alle Paare, `null` heißt nicht routbar.

```js
// Matrix: Fahrzeiten zwischen mehreren Orten in einer Anfrage.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const ORTE = { Melk: [15.33, 48.228], 'St. Pölten': [15.626, 48.204], Krems: [15.6, 48.41], Amstetten: [14.87, 48.123] };
const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [15.3, 48.25], zoom: 8 });
map.addControl(new maplibregl.NavigationControl());
for (const [name, p] of Object.entries(ORTE)) new maplibregl.Marker().setLngLat(p).setPopup(new maplibregl.Popup().setText(name)).addTo(map);

document.getElementById('los').addEventListener('click', async () => {
  const tabelle = document.getElementById('ergebnis'), namen = Object.keys(ORTE);
  try {
    const koordinaten = Object.values(ORTE).map((p) => p.join(',')).join(';');
    const r = await fetch(`https://api.mapx.at/v1/matrix/driving/${koordinaten}`);
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const { durations } = await r.json(); // Sekunden, null = nicht routbar
    tabelle.innerHTML = '<tr><th></th>' + namen.map((n) => `<th>${n}</th>`).join('') + '</tr>' +
      durations.map((zeile, i) => `<tr><th>${namen[i]}</th>` + zeile.map((s) => `<td>${s == null ? '–' : Math.round(s / 60) + ' min'}</td>`).join('') + '</tr>').join('');
  } catch (err) { tabelle.textContent = 'Fehler: ' + err.message; }
});
```

## Verwendete APIs

- [Routing](/api/routing/#matrix) `GET api.mapx.at/v1/matrix/{profile}/{coordinates}`
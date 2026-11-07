# Fahrzeitmatrix

Fahrzeiten zwischen mehreren Orten in einer Anfrage.

Die Matrix-API nimmt Quellen (und optional Ziele) und liefert Dauer und Distanz für alle Paare, `null` heißt nicht routbar.

> Such- und Routing-Dienst antworten nur auf freigegebene Herkünfte (CORS). Die Beispiele laufen hier, sobald `docs.mapx.at` freigegeben ist; auf eigenen Seiten braucht es ebenfalls eine Freigabe der Herkunft.

```js
// Matrix: Fahrzeiten zwischen mehreren Orten in einer Anfrage.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const ORTE = { Melk: [15.33, 48.228], 'St. Pölten': [15.626, 48.204], Krems: [15.6, 48.41], Amstetten: [14.87, 48.123] };
const map = new maplibregl.Map({ container: 'map', style: 'https://mapx.at/styles/standard-tag-normal.json', center: [15.3, 48.25], zoom: 8 });
map.addControl(new maplibregl.NavigationControl());
for (const [name, p] of Object.entries(ORTE)) new maplibregl.Marker().setLngLat(p).setPopup(new maplibregl.Popup().setText(name)).addTo(map);

document.getElementById('los').addEventListener('click', async () => {
  const tabelle = document.getElementById('ergebnis'), namen = Object.keys(ORTE);
  try {
    const r = await fetch('https://routing.mapx.at/matrix', { // künftig /v1/matrix
      method: 'POST', headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ profile: 'car', sources: Object.values(ORTE) }),
    });
    if (!r.ok) throw new Error('HTTP ' + r.status);
    const { durations } = await r.json(); // Sekunden, null = nicht routbar
    tabelle.innerHTML = '<tr><th></th>' + namen.map((n) => `<th>${n}</th>`).join('') + '</tr>' +
      durations.map((zeile, i) => `<tr><th>${namen[i]}</th>` + zeile.map((s) => `<td>${s == null ? '–' : Math.round(s / 60) + ' min'}</td>`).join('') + '</tr>').join('');
  } catch (err) { tabelle.textContent = 'Fehler: ' + err.message; }
});
```

## Verwendete APIs

- Routing `POST routing.mapx.at/matrix`

> Aktuell `routing.mapx.at/matrix`; `POST /v1/matrix` ist geplant.
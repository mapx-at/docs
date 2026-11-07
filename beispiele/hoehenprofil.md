# Höhenprofil

Linie klicken, Profil sehen.

Die Stützpunkte gehen als `coordinates` an `/v1/elevation/profile`, das `samples` gleichmäßig verteilte Messpunkte samt Auf- und Abstieg liefert. Ein Canvas zeichnet das Profil.

```js
// Höhenprofil: Linie in die Karte klicken, /v1/elevation/profile liefert die Höhen, ein Canvas zeichnet das Profil.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/gelaende', center: [12.70, 47.08], zoom: 12 });
map.addControl(new maplibregl.NavigationControl());
let punkte = [[12.67, 47.065], [12.71, 47.09]];

map.on('load', () => {
  map.addSource('linie', { type: 'geojson', data: { type: 'FeatureCollection', features: [] } });
  map.addLayer({ id: 'linie', type: 'line', source: 'linie', paint: { 'line-color': '#2346d8', 'line-width': 4 } });
  zeichne();
});
map.on('click', (e) => { punkte.push([e.lngLat.lng, e.lngLat.lat]); zeichne(); });
document.getElementById('neu').addEventListener('click', () => { punkte = []; zeichne(); });

async function zeichne() {
  map.getSource('linie').setData({ type: 'Feature', geometry: { type: 'LineString', coordinates: punkte } });
  const canvas = document.getElementById('profil'), g = canvas.getContext('2d');
  g.clearRect(0, 0, canvas.width, canvas.height);
  if (punkte.length < 2) { document.getElementById('info').textContent = 'Mindestens zwei Punkte setzen.'; return; }
  const pfad = punkte.map((p) => p.map((v) => v.toFixed(5)).join(',')).join(';');
  const antwort = await fetch(`https://api.mapx.at/v1/elevation/profile?coordinates=${pfad}&samples=120`);
  const { distances, elevations } = (await antwort.json()).properties;
  const pts = distances.map((d, i) => ({ d, h: elevations[i] })).filter((p) => p.h != null);
  const dMax = pts.at(-1).d, hMin = Math.min(...pts.map((p) => p.h)), hMax = Math.max(...pts.map((p) => p.h));
  const W = canvas.width, H = canvas.height, x = (d) => 50 + (d / dMax) * (W - 60), y = (h) => H - 24 - ((h - hMin) / (hMax - hMin || 1)) * (H - 40);
  g.beginPath(); pts.forEach((p, i) => (i ? g.lineTo(x(p.d), y(p.h)) : g.moveTo(x(p.d), y(p.h))));
  g.strokeStyle = '#2346d8'; g.lineWidth = 2; g.stroke();
  g.fillStyle = '#6b7079'; g.font = '12px sans-serif';
  g.fillText(`${Math.round(hMax)} m`, 4, 16); g.fillText(`${Math.round(hMin)} m`, 4, H - 28); g.fillText(`${(dMax / 1000).toFixed(2)} km`, W - 60, H - 6);
  let auf = 0, ab = 0; pts.forEach((p, i) => { if (i) { const dh = p.h - pts[i - 1].h; dh > 0 ? (auf += dh) : (ab -= dh); } });
  document.getElementById('info').textContent = `${(dMax / 1000).toFixed(2)} km, Aufstieg ${Math.round(auf)} m, Abstieg ${Math.round(ab)} m`;
}
```

## Verwendete APIs

- [Höhe und Profil](/api/hoehe/#profil-endpunkt) `GET api.mapx.at/v1/elevation/profile`
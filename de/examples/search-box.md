# Search Box mit Vorschlägen

Vorschläge beim Tippen, Position nach dem Klick, Typfilter.

Die Search Box arbeitet in zwei Schritten: `suggest` liefert Vorschläge beim Tippen, `retrieve` die Position des gewählten. Für beide Aufrufe dasselbe `session_token` verwenden, für die nächste Suche ein neues. Der Typfilter begrenzt die Vorschläge.

```js
// Search Box: suggestions while typing (suggest), position and details after the click (retrieve). One session token per search.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [14.3, 47.6], zoom: 7 });
map.addControl(new maplibregl.NavigationControl());
const input = document.getElementById('q'), types = document.getElementById('types'), list = document.getElementById('treffer');
let marker, token = crypto.randomUUID();

input.addEventListener('input', async () => {
  list.replaceChildren();
  if (input.value.trim().length < 2) return;
  const c = map.getCenter();
  const url = `https://api.mapx.at/v1/searchbox/suggest?q=${encodeURIComponent(input.value)}&session_token=${token}&proximity=${c.lng},${c.lat}&limit=6` + (types.value ? `&types=${types.value}` : '');
  try {
    const res = await fetch(url);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    for (const s of (await res.json()).suggestions) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = `${s.name} (${s.feature_type}) ${s.place_formatted ?? ''}`;
      b.addEventListener('click', () => pick(s.mapbox_id));
      li.append(b); list.append(li);
    }
  } catch (e) { list.textContent = 'Error: ' + e.message; }
});

async function pick(id) {
  const res = await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${encodeURIComponent(id)}?session_token=${token}`);
  const f = (await res.json()).features[0];
  marker?.remove();
  marker = new maplibregl.Marker().setLngLat(f.geometry.coordinates).addTo(map);
  map.flyTo({ center: f.geometry.coordinates, zoom: f.properties.zoom ?? 15 });
  list.replaceChildren();
  token = crypto.randomUUID(); // the session ends with retrieve
}
```

## Verwendete APIs

- [Search Box](/de/api/search-box/) `GET api.mapx.at/v1/searchbox/suggest`
- [Search Box](/de/api/search-box/) `GET api.mapx.at/v1/searchbox/retrieve/{id}`
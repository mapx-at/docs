# Suche

Orte, Adressen und Gipfel suchen und anspringen.

Die Such-API ist ein schreibgeschützter Meilisearch-Index mit einem öffentlichen Search-only-Key. Treffer enthalten Koordinate und Umhüllende zum Zoomen.

> Such- und Routing-Dienst antworten nur auf freigegebene Herkünfte (CORS). Die Beispiele laufen hier, sobald `docs.mapx.at` freigegeben ist; auf eigenen Seiten braucht es ebenfalls eine Freigabe der Herkunft.

```js
// Suche: Orte, Adressen, Gipfel und Grundstücke über die Such-API (Meilisearch, nur lesend).
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const SUCHE = 'https://search.mapx.at/indexes/places/search';
const KEY = '<SEARCH_KEY>'; // öffentlicher Search-only-Key, darf im Browser stehen
const map = new maplibregl.Map({ container: 'map', style: 'https://mapx.at/styles/standard-tag-normal.json', center: [14.3, 47.6], zoom: 7 });
map.addControl(new maplibregl.NavigationControl());
let marker;

const eingabe = document.getElementById('q'), liste = document.getElementById('treffer');
eingabe.addEventListener('input', async () => {
  const q = eingabe.value.trim();
  liste.replaceChildren();
  if (q.length < 2) return;
  try {
    const r = await fetch(SUCHE, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json', Authorization: 'Bearer ' + KEY },
      body: JSON.stringify({ q, limit: 6, filter: 'type != parcel' }),
    });
    for (const h of (await r.json()).hits) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = h.label + (h.context ? ' – ' + h.context : '');
      b.addEventListener('click', () => {
        marker?.remove();
        marker = new maplibregl.Marker().setLngLat([h._geo.lng, h._geo.lat]).addTo(map);
        h.bbox ? map.fitBounds([[h.bbox[0], h.bbox[1]], [h.bbox[2], h.bbox[3]]], { padding: 60, maxZoom: 17 }) : map.flyTo({ center: [h._geo.lng, h._geo.lat], zoom: h.zoom ?? 14 });
        liste.replaceChildren();
      });
      li.append(b); liste.append(li);
    }
  } catch (e) { liste.textContent = 'Fehler: ' + e.message; }
});
```

## Verwendete APIs

- Suche `POST search.mapx.at/indexes/places/search`

> Das Beispiel nutzt direkt den Suchindex. `GET /v1/geocode` und `/v1/reverse` (Mapbox-ähnliche GeoJSON-Antwort) sind geplant, die Einbindung wechselt dann nur die URL.
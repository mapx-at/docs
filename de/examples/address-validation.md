# Adressvalidierung

Eine Adresse gegen das amtliche Register prüfen und das korrigierte Ergebnis zeigen.

Die Validierung gleicht die vier Adressteile mit dem Adressregister des BEV ab. Das Ergebnis hat einen `status` (`match`, `corrected`, `ambiguous`, `not_found`), die Liste der `deviations` und entweder die passende `address` mit Koordinaten oder eine Liste von `candidates`. Die vorbelegte Adresse enthält absichtlich einen Tippfehler.

```js
// Address validation: check a typed address against the official register (BEV) and show the corrected result.
maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [13.8, 47.7], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
const out = document.getElementById('ergebnis');
let marker;

document.getElementById('pruefen').addEventListener('click', async () => {
  const p = new URLSearchParams();
  for (const k of ['street', 'housenumber', 'postcode', 'place']) p.set(k, document.getElementById(k).value);
  try {
    const res = await fetch('https://api.mapx.at/v1/address/validate?' + p);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    const r = await res.json(); // status: match, corrected, ambiguous, not_found
    const hit = r.address ? { text: r.address, at: r.coordinates } : r.candidates?.[0] && { text: r.candidates[0].full_address, at: r.candidates[0].coordinates };
    out.textContent = `Status: ${r.status}` + (r.deviations?.length ? `, deviations: ${r.deviations.map((d) => `${d.field} ${d.input} > ${d.expected}`).join('; ')}` : '') + (hit ? `, ${r.address ? 'address' : 'first candidate'}: ${hit.text}` : '');
    marker?.remove();
    if (hit) { marker = new maplibregl.Marker().setLngLat(hit.at).addTo(map); map.flyTo({ center: hit.at, zoom: 16 }); }
  } catch (e) { out.textContent = 'Error: ' + e.message; }
});
```

## Verwendete APIs

- [Adressen](/de/api/address/) `GET api.mapx.at/v1/address/validate`
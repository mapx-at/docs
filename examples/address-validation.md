# Address validation

Check an address against the official register and show the corrected result.

Validation compares the four address parts with the BEV address register. The result has a `status` (`match`, `corrected`, `ambiguous`, `not_found`), the list of `deviations` and either the matching `address` with coordinates or a list of `candidates`. The pre-filled address has a typo on purpose.

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

## APIs used

- [Address](/api/address/) `GET api.mapx.at/v1/address/validate`
# Address autofill in a form

Type a street, pick a suggestion, the form fills itself.

Autofill is made for checkout and registration forms: one text field, one suggestion list, and the other fields (house number, postcode, place) are filled from the chosen suggestion. Each suggestion already carries its coordinates.

```js
// Address autofill: type a street and number, pick a suggestion, the form fields fill themselves.
const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at
// Adds the token to every request MapLibre makes to mapx.at (style, tiles, fonts, sprites), never to other hosts.
const transformRequest = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes('?') ? '&' : '?') + 'access_token=' + TOKEN : url });
const AUTH = { headers: { Authorization: 'Bearer ' + TOKEN } };

maplibregl.addProtocol('pmtiles', new pmtiles.Protocol().tile);

const map = new maplibregl.Map({ transformRequest, container: 'map', style: 'https://api.mapx.at/v1/styles/streets', center: [13.8, 47.7], zoom: 6.5 });
map.addControl(new maplibregl.NavigationControl());
const f = Object.fromEntries(['street', 'housenumber', 'postcode', 'place'].map((k) => [k, document.getElementById(k)]));
const list = document.getElementById('treffer');
let marker;

f.street.addEventListener('input', async () => {
  list.replaceChildren();
  if (f.street.value.trim().length < 3) return;
  try {
    const res = await fetch(`https://api.mapx.at/v1/autofill/suggest?q=${encodeURIComponent(f.street.value)}&limit=5&session_token=demo`, AUTH);
    if (!res.ok) throw new Error('HTTP ' + res.status);
    for (const s of (await res.json()).suggestions) {
      const li = document.createElement('li'), b = document.createElement('button');
      b.type = 'button'; b.textContent = s.full_address;
      b.addEventListener('click', () => {
        for (const k of Object.keys(f)) f[k].value = s[k] ?? '';
        marker?.remove();
        marker = new maplibregl.Marker().setLngLat(s.coordinates).addTo(map);
        map.flyTo({ center: s.coordinates, zoom: 16 });
        list.replaceChildren();
      });
      li.append(b); list.append(li);
    }
  } catch (e) { list.textContent = 'Error: ' + e.message; }
});
```

## APIs used

- [Address](/api/address/) `GET api.mapx.at/v1/autofill/suggest`
# Search

Geocoding, type-ahead search, address autofill and validation.

Pick the endpoint by what the user does.

| Task | Endpoint |
|---|---|
| Convert a typed address or place to coordinates | [`/v1/geocode`](/api/geocoding/) |
| Show suggestions while the user types | [`/v1/searchbox/suggest`](/api/search-box/) |
| Fill a postal address form | [`/v1/autofill/suggest`](/api/address/) |
| Check an address | [`/v1/address/validate`](/api/address/) |
| Find the address at a point | [`/v1/reverse`](/api/geocoding/) |
| Show places of a kind (pharmacy, station) | [`/v1/searchbox/category/{category}`](/api/search-box/) |

All data is official: the address register, place names, administrative boundaries and cadastre from BEV. Requests are in the `search` zone, 10 per second.

## Geocode an address

```js
const url = "https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&types=address&limit=3";
const { features } = await (await fetch(url)).json();
const [lon, lat] = features[0].center;
```

Always pass `types` (`address`, `street`, `locality`, `place`, `district`, `region`, `postcode`, `poi`). Narrow the result with `bbox=minlon,minlat,maxlon,maxlat` or rank by distance with `proximity=lon,lat`.

## Reverse geocode

```bash
curl -s "https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&types=address&limit=1" | jq '.features[0].place_name'
```

## Type-ahead search

Call `suggest` on every pause in typing (debounce 200 ms), then `retrieve` once for the chosen entry. Send the same `session_token` for both calls.

```js
const token = crypto.randomUUID();
const s = await (await fetch(`https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=5&session_token=${token}`)).json();
const id = s.suggestions[0].mapbox_id;
const r = await (await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${id}?session_token=${token}`)).json();
const [lon, lat] = r.features[0].geometry.coordinates;
```

Suggestions carry a category (`poi_category_ids`, for example `place_of_worship_church`) and the administrative context. Cancel the previous request when a new key is typed (`AbortController`). Example: [Search](/examples/search/).

## Places by category

```bash
curl -s "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq ".listItems[0:3]"
curl -s "https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=16.3725,48.2085&limit=5"
```

`list/category` returns all category IDs with localized names (`language=de` or `en`).

## Address forms

Autofill returns split fields that map to form inputs.

```js
const a = await (await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3")).json();
const { address_line1, postcode, place, coordinates } = a.suggestions[0];
```

Validation checks what the user typed:

```bash
curl -s "https://api.mapx.at/v1/address/validate?q=Hauptstr.%2012a,%203390" | jq '{status, confidence, address}'
```

`status` is `match`, `corrected`, `ambiguous` (then `candidates[]`) or `not_found`. Abbreviations, umlauts and `ß` are understood. Use POST with JSON for structured input.

## Batch

`POST /v1/geocode/batch` geocodes up to 1000 queries in one request (zone `heavy`, 30 per minute).

## Next

- [Geocoding API](/api/geocoding/), [Search Box API](/api/search-box/), [Address API](/api/address/)
- [Parcels](/guides/parcels/): search results of type parcel
- [Errors and limits](/api/errors/)
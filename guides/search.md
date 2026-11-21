# Address search: geocoding and autofill

Source: https://docs.mapx.at/guides/search/

> Add address and place search for Austria to your app: geocoding, type-ahead Search Box, address autofill for forms and address validation.

Pick the endpoint by what the user does.

| Task | Endpoint |
|---|---|
| Convert a typed address or place to coordinates | [`/v1/geocode`](https://docs.mapx.at/api/geocoding.md) |
| Show suggestions while the user types | [`/v1/searchbox/suggest`](https://docs.mapx.at/api/search-box.md) |
| Fill a postal address form | [`/v1/autofill/suggest`](https://docs.mapx.at/api/address.md) |
| Check an address | [`/v1/address/validate`](https://docs.mapx.at/api/address.md) |
| Find the address at a point | [`/v1/reverse`](https://docs.mapx.at/api/geocoding.md) |
| Show places of a kind (fuel station, station) | [`/v1/searchbox/category/{category}`](https://docs.mapx.at/api/search-box.md) |

All data is official: the address register, place names, administrative boundaries from BEV. Requests are in the `search` zone, 10 per second and need a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); the samples send it in the `Authorization` header, which browsers may send too.

## Geocode an address

```js
const url = "https://api.mapx.at/v1/geocode?q=Herrengasse%2013,%201010%20Wien&types=address&limit=3";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
const [lon, lat] = features[0].center;
```

`features` can be empty; check the length before you read `features[0]`. Always pass `types` (`address`, `street`, `locality`, `place`, `district`, `region`, `postcode`, `poi`, `road_ref` for road kilometers such as `A1 km 41`). Narrow the result with `bbox=minlon,minlat,maxlon,maxlat` or rank by distance with `proximity=lon,lat`.

## Reverse geocode

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&types=address&limit=1" | jq '.features[0].place_name'
```

## Type-ahead search

Call `suggest` on every pause in typing (debounce 200 ms), then `retrieve` once for the chosen entry. Send the same `session_token` for both calls.

```js
const token = crypto.randomUUID();
const s = await (await fetch(`https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=5&session_token=${token}`, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
const id = s.suggestions[0].mapbox_id;
const r = await (await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${id}?session_token=${token}`, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
const [lon, lat] = r.features[0].geometry.coordinates;
```

Suggestions carry a category (`poi_category_ids`, for example `place_of_worship_church`) and the administrative context. Cancel the previous request when a new key is typed (`AbortController`). Example: [Search](https://docs.mapx.at/examples/search.md).

## Places by category

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq ".listItems[0:3]"
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=5"
```

`list/category` returns all category IDs with localized names (`language=de` or `en`).

## Address forms

Autofill returns split fields that map to form inputs.

```js
const a = await (await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013,%201010%20Wien&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
const { address_line1, postcode, place, coordinates } = a.suggestions[0];
```

Validation checks what the user typed:

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk" | jq '{status, confidence, address}'
```

`status` is `match`, `corrected`, `ambiguous` (then `candidates[]`) or `not_found`. Abbreviations, umlauts and `ß` are understood. Use POST with JSON for structured input.

## Batch

`POST /v1/geocode/batch` geocodes up to 1000 queries in one request (zone `heavy`, 30 per minute). Batch geocoding needs a secret token (`mx_sk_…`) and therefore belongs on a server; with a public token the API answers `403` `pk-not-allowed`.

## Next

- [Geocoding API](https://docs.mapx.at/api/geocoding.md), [Search Box API](https://docs.mapx.at/api/search-box.md), [Address API](https://docs.mapx.at/api/address.md)
- [Errors and limits](https://docs.mapx.at/api/errors.md)

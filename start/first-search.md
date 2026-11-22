# First search: geocoding request

Source: https://docs.mapx.at/start/first-search/

> Find an address or place in Austria with one geocoding request (curl, JavaScript, Python), then show the result as a marker on a map.

## Request

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1" | jq '.features[0] | {name: .properties.name, place_formatted: .properties.place_formatted, coordinates: .geometry.coordinates}'
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const { features } = await res.json();
console.log(features[0].properties.name, features[0].geometry.coordinates); // coordinates are [lon, lat]
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode", params={"q": "Stephansplatz 1, Wien", "types": "address", "limit": 1}, headers={"Authorization": "Bearer YOUR_TOKEN"})
feature = res.json()["features"][0]
print(feature["properties"]["name"], feature["properties"]["place_formatted"], feature["geometry"]["coordinates"])
```

The response is a GeoJSON `FeatureCollection`; the `curl` example returns, for example, `{"name":"Stephansplatz 1","place_formatted":"Innere Stadt, 1010 Wien, W","coordinates":[16.372605,48.208728]}`. On `401` or `403` see [Access and limits](https://docs.mapx.at/start/access-and-limits.md). Each feature has an `id`, a point `geometry` (`[lon, lat]`) and in `properties` the fields `type`, `name`, `place_formatted`, `relevance`, `accuracy` and `address` (street, house number, postcode, place … separately); Search Box and Autofill answer in the same format. `types=address` limits the results to addresses; without it a street or place can come first even for a house number.

## Show the result on the map

Add this to the page from [First map](https://docs.mapx.at/start/first-map.md) after `new maplibregl.Map(...)` (store the map in `const map`). It belongs in the `<script type="module">`; top-level `await` works only there:

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const [hit] = (await res.json()).features;
new maplibregl.Marker().setLngLat(hit.geometry.coordinates).addTo(map);
map.flyTo({ center: hit.geometry.coordinates, zoom: 16 });
```

## Next

- [Search guide](https://docs.mapx.at/guides/search.md): type-ahead, categories, address forms
- [Geocoding API](https://docs.mapx.at/api/geocoding.md)

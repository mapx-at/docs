# First search: geocoding request

Source: https://docs.mapx.at/start/first-search/

> Find an address or place in Austria with one geocoding request (curl, JavaScript, Python), then show the result as a marker on a map.

## Request

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1" | jq '.features[0] | {place_name, center}'
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1&access_token=YOUR_TOKEN");
const { features } = await res.json();
console.log(features[0].place_name, features[0].center); // center is [lon, lat]
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode", params={"q": "Stephansplatz 1, Wien", "types": "address", "limit": 1}, headers={"Authorization": "Bearer YOUR_TOKEN"})
feature = res.json()["features"][0]
print(feature["place_name"], feature["center"])
```

The response is a GeoJSON `FeatureCollection`. Each feature has `place_name`, `center` (`lon,lat`) and `geometry`. `types=address` limits the results to addresses; without it a street or place can come first even for a house number.

## Show the result on the map

Add this to the page from [First map](https://docs.mapx.at/start/first-map.md) after `new maplibregl.Map(...)` (store the map in `const map`):

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1&access_token=YOUR_TOKEN");
const [hit] = (await res.json()).features;
new maplibregl.Marker().setLngLat(hit.center).addTo(map);
map.flyTo({ center: hit.center, zoom: 16 });
```

Put the code in a `<script type="module">` so that top-level `await` works.

## Next

- [Search guide](https://docs.mapx.at/guides/search.md): type-ahead, categories, address forms
- [Geocoding API](https://docs.mapx.at/api/geocoding.md)

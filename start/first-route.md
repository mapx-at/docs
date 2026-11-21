# First route: routing request

Source: https://docs.mapx.at/start/first-route/

> Request a driving route from Vienna to Melk with the mapx routing API (curl, JavaScript, Python) and draw the result as a line on a map.

## Request

Coordinates are always `lon,lat`, separated by `;`.

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=false" \
  | jq '.routes[0] | {distance, duration}'
```

**JavaScript**

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const { routes } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
console.log(routes[0].distance / 1000, "km", routes[0].duration / 60, "min");
```

**Python**

```python
import requests

url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272"
route = requests.get(url, params={"overview": "simplified"}, headers={"Authorization": "Bearer YOUR_TOKEN"}).json()["routes"][0]
print(route["distance"] / 1000, "km", route["duration"] / 60, "min")
```

The response of the `curl` example looks like this: `{"distance":90076.3,"duration":3649.0}`. `distance` is in meters, `duration` in seconds. On `401` or `403` see [Access and limits](https://docs.mapx.at/start/access-and-limits.md). Other profiles: `driving-traffic` (avoids reported closures), `driving-truck`, `cycling`, `cycling-mtb` and `walking`; parameters per profile in the [Directions reference](https://docs.mapx.at/api/directions.md). See the [Navigation guide](https://docs.mapx.at/guides/navigation.md).

## Draw it on the map

With the map from [First map](https://docs.mapx.at/start/first-map.md) in `map`:

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const route = (await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json()).routes[0];
const draw = () => {
  map.addSource("route", { type: "geojson", data: { type: "Feature", geometry: route.geometry } });
  map.addLayer({ id: "route", type: "line", source: "route", paint: { "line-color": "#1d5bd8", "line-width": 5 } });
  const bounds = new maplibregl.LngLatBounds();
  route.geometry.coordinates.forEach((c) => bounds.extend(c));
  map.fitBounds(bounds, { padding: 40 });
};
map.loaded() ? draw() : map.once("load", draw); // the map may already be loaded when the route arrives
```

## Next

- [Navigation guide](https://docs.mapx.at/guides/navigation.md): truck, bike, tolls, alternatives
- [Directions API](https://docs.mapx.at/api/directions.md)

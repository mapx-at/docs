# First route

Request a driving route from Vienna to Melk and draw it on a map.

## Request

Coordinates are always `lon,lat`, separated by `;`.

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=false" \
  | jq '.routes[0] | {distance, duration}'
```

</TabItem>
<TabItem label="JavaScript">

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const { routes } = await (await fetch(url)).json();
console.log(routes[0].distance / 1000, "km", routes[0].duration / 60, "min");
```

</TabItem>
<TabItem label="Python">

```python

url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272"
route = requests.get(url, params={"overview": "simplified"}).json()["routes"][0]
print(route["distance"] / 1000, "km", route["duration"] / 60, "min")
```

</TabItem>
</Tabs>

`distance` is in meters, `duration` in seconds. Other profiles: `driving-truck`, `cycling`, `walking`. See the [Navigation guide](/guides/navigation/).

## Draw it on the map

With the map from [First map](/start/first-map/) in `map`:

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const route = (await (await fetch(url)).json()).routes[0];
map.on("load", () => {
  map.addSource("route", { type: "geojson", data: { type: "Feature", geometry: route.geometry } });
  map.addLayer({ id: "route", type: "line", source: "route", paint: { "line-color": "#1d5bd8", "line-width": 5 } });
  map.fitBounds([[15.3294, 48.2085], [16.3725, 48.2272]], { padding: 40 });
});
```

## Next

- [Navigation guide](/guides/navigation/): truck, bike, tolls, alternatives
- [Directions API](/api/directions/)
# Erste Route


## Anfrage

Koordinaten sind immer `lon,lat`, getrennt durch `;`.

**curl**


```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=false" \
  | jq '.routes[0] | {distance, duration}'
```

**JavaScript**


```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const { routes } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(routes[0].distance / 1000, "km", routes[0].duration / 60, "min");
```

**Python**


```python
import requests

url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272"
route = requests.get(url, params={"overview": "simplified"}, headers={"Authorization": "Bearer DEIN_TOKEN"}).json()["routes"][0]
print(route["distance"] / 1000, "km", route["duration"] / 60, "min")
```


`distance` ist in Metern, `duration` in Sekunden. Weitere Profile: `driving-truck`, `cycling`, `walking`. Siehe den [Navigation-Guide](/de/guides/navigation/).

## Auf der Karte zeichnen

Mit der Karte aus [Erste Karte](/de/start/first-map/) in `map`:

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const route = (await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).routes[0];
map.on("load", () => {
  map.addSource("route", { type: "geojson", data: { type: "Feature", geometry: route.geometry } });
  map.addLayer({ id: "route", type: "line", source: "route", paint: { "line-color": "#1d5bd8", "line-width": 5 } });
  map.fitBounds([[15.3294, 48.2085], [16.3725, 48.2272]], { padding: 40 });
});
```

## Nächste Schritte

- [Navigation-Guide](/de/guides/navigation/): Lkw, Rad, Maut, Alternativen
- [Directions API](/de/api/directions/)


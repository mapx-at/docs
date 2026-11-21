# Erste Route: Routing-Anfrage

Source: https://docs.mapx.at/de/start/first-route/

> Eine Autoroute von Wien nach Melk mit der mapx-Routing-API anfordern (curl, JavaScript, Python) und das Ergebnis als Linie auf einer Karte zeichnen.

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

Die Antwort des `curl`-Beispiels sieht so aus: `{"distance":90076.3,"duration":3649.0}`. `distance` ist in Metern, `duration` in Sekunden. Bei `401` oder `403` siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Weitere Profile: `driving-traffic` (umfährt gemeldete Sperren), `driving-truck`, `cycling`, `cycling-mtb` und `walking`; Parameter je Profil in der [Directions-Referenz](https://docs.mapx.at/de/api/directions.md). Siehe den [Navigation-Guide](https://docs.mapx.at/de/guides/navigation.md).

## Auf der Karte zeichnen

Mit der Karte aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in `map`:

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?overview=simplified";
const route = (await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).routes[0];
const draw = () => {
  map.addSource("route", { type: "geojson", data: { type: "Feature", geometry: route.geometry } });
  map.addLayer({ id: "route", type: "line", source: "route", paint: { "line-color": "#1d5bd8", "line-width": 5 } });
  const bounds = new maplibregl.LngLatBounds();
  route.geometry.coordinates.forEach((c) => bounds.extend(c));
  map.fitBounds(bounds, { padding: 40 });
};
map.loaded() ? draw() : map.once("load", draw); // die Karte kann beim Eintreffen der Route schon geladen sein
```

## Nächste Schritte

- [Navigation-Guide](https://docs.mapx.at/de/guides/navigation.md): Lkw, Rad, Maut, Alternativen
- [Directions API](https://docs.mapx.at/de/api/directions.md)

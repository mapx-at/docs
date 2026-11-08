# Departures at a stop

Click near a stop, see the next departures.

`DeparturesPopup` from the [SDK](/guides/sdk/) finds the nearest stop to a click and lists the next departures from the scheduled timetable (no real-time data). The dots are the stops of the visible area from `/v1/stops`.

```js
// Departures: click near a stop to see the next departures. Uses the DeparturesPopup of the mapx-gl SDK.
import mapx from "mapx-gl-sdk";

const TOKEN = 'YOUR_TOKEN'; // your public token (mx_pk_…) from https://account.mapx.at

const map = mapx.map({ accessToken: TOKEN, container: "map", style: "streets", center: [16.3725, 48.2085], zoom: 15.5 });
new mapx.DeparturesPopup(map, { limit: 8, radius: 150 });

// Show the stops of the visible area as dots.
map.on("load", () => {
  map.addSource("stops", { type: "geojson", data: { type: "FeatureCollection", features: [] } });
  map.addLayer({ id: "stops", type: "circle", source: "stops", paint: { "circle-radius": 6, "circle-color": "#2346d8", "circle-stroke-color": "#fff", "circle-stroke-width": 2 } });
  const load = async () => {
    const b = map.getBounds();
    const list = await mapx.stops({ bbox: [b.getWest(), b.getSouth(), b.getEast(), b.getNorth()], limit: 100 });
    map.getSource("stops").setData({ type: "FeatureCollection", features: list.map((s) => ({ type: "Feature", properties: { name: s.name }, geometry: { type: "Point", coordinates: s.location } })) });
  };
  map.on("moveend", load);
  load();
});
```

## APIs used

- [Transit](/api/transit/) `GET api.mapx.at/v1/stops`
- [Transit](/api/transit/) `GET api.mapx.at/v1/departures`
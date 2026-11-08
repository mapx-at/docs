# Abfahrten an einer Haltestelle

Nahe an eine Haltestelle klicken, nächste Abfahrten sehen.

`DeparturesPopup` aus dem [SDK](/de/guides/sdk/) sucht die nächste Haltestelle zum Klick und listet die nächsten Abfahrten aus dem Soll-Fahrplan (keine Echtzeitdaten). Die Punkte sind die Haltestellen im sichtbaren Ausschnitt aus `/v1/stops`.

```js
// Departures: click near a stop to see the next departures. Uses the DeparturesPopup of the mapx-gl SDK.
import mapx from "mapx-gl-sdk";

const map = mapx.map({ container: "map", style: "streets", center: [16.3725, 48.2085], zoom: 15.5 });
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

## Verwendete APIs

- [Öffentlicher Verkehr](/de/api/transit/) `GET api.mapx.at/v1/stops`
- [Öffentlicher Verkehr](/de/api/transit/) `GET api.mapx.at/v1/departures`
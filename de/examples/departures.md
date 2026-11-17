# Abfahrten an einer Haltestelle: Beispiel

Source: https://docs.mapx.at/de/examples/departures/

> Nahe an eine ÖV-Haltestelle in Österreich klicken und die nächsten Abfahrten sehen: ÖV-Abfahrts-API mit Live-Karte und lauffähigem Code.

`DeparturesPopup` aus dem [SDK](https://docs.mapx.at/de/guides/sdk/index.md) sucht die nächste Haltestelle zum Klick und listet die nächsten Abfahrten aus dem Soll-Fahrplan (keine Echtzeitdaten). Die Punkte sind die Haltestellen im sichtbaren Ausschnitt aus `/v1/stops`.

## Code

````js
// Departures: click near a stop to see the next departures. Uses the DeparturesPopup of the mapx-gl SDK.
import mapx from "https://map.mapx.at/sdk/v1/mapx-gl.bundle.js";

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
````

## Verwendete APIs

- [Öffentlicher Verkehr](https://docs.mapx.at/de/api/transit.md) `GET api.mapx.at/v1/stops`
- [Öffentlicher Verkehr](https://docs.mapx.at/de/api/transit.md) `GET api.mapx.at/v1/departures`

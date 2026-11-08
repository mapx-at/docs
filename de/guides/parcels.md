# Grundstücke

Das Katastergrundstück an einem Punkt finden, Grundstücksgrenzen zeichnen und Attribute lesen.

Der Kataster (BEV, DKM) steht als Vektor-Tileset zum Zeichnen und als API für das Grundstück an einem Punkt bereit. Die Daten dienen der Orientierung und sind nicht rechtsverbindlich.

| Aufgabe | Verwendung |
|---|---|
| Grundstücksgrenzen zeichnen | Tileset `cadastre` (z16) |
| Grundstücksnummer, Fläche und Katastralgemeinde an einem Punkt | [`/v1/parcel/{lon},{lat}`](/de/api/tilequery/) |
| Ein beliebiges Vektor-Tileset an einem Punkt abfragen | [`/v1/tilequery/{tilesets}/{lon},{lat}.json`](/de/api/tilequery/) |
| Ein Grundstück suchen | [Geocoding](/de/guides/search/): Grundstücke gehören zu den durchsuchten Objekten |

## Grundstück an einem Punkt

```bash
curl -s "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_number, cadastral_municipality_id, area_m2, is_building_land}'
```

Das Ergebnis ist eine `FeatureCollection` mit dem Grundstückspolygon. Die Fläche wird aus den Grenzlinien berechnet und weicht um Promille vom Registerwert ab. Ein Gebäude liefert sein Baugrundstück; die Nummer beginnt dann mit einem Punkt. Grundstücke, die über das 2-km-Tilefenster um den Punkt hinausreichen, fehlen, und außerhalb der Katasterdaten ist die Liste leer.

## Auf der Karte ein Grundstück anklicken

```js
map.on("load", () => {
  map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
  map.addLayer({ id: "parcel-lines", type: "line", source: "cadastre", "source-layer": "parcel_l", paint: { "line-color": "#e11d48" } });
});
map.on("click", async (e) => {
  const { lng, lat } = e.lngLat;
  const res = await fetch(`https://api.mapx.at/v1/parcel/${lng},${lat}`);
  const [parcel] = (await res.json()).features;
  if (!parcel) return;
  const p = parcel.properties;
  new maplibregl.Popup().setLngLat(e.lngLat)
    .setHTML(`Parcel ${p.parcel_number}<br>${Math.round(p.area_m2)} m²`).addTo(map);
});
```

Vollständiges Beispiel: [Grundstück per Klick](/de/examples/parcel-on-click/). Die Katastralgemeinde (`cadastral_municipality_id`) gehört zur Identität eines Grundstücks: Die Nummer allein ist nicht eindeutig.

## Tilequery

Tilequery liest das Vektor-Tile im höchsten Zoom und liefert die Features, die dem Punkt am nächsten liegen (`radius` bis 2000 m).

```bash
curl -s "https://api.mapx.at/v1/tilequery/cadastre/15.3294,48.2272.json?radius=30&limit=3" | jq '.features[].properties.tilequery'
```

Tilesets lassen sich mit Komma kombinieren, zum Beispiel `streets,cadastre`. Jedes Feature hat `tilequery.distance` (m), `geometry` und `layer`.

## Grundstücksbild für ein Inserat

Das Grundstück mit einem [Static Image](/de/guides/static-images/) kombinieren und das Polygon als GeoJSON-Overlay verwenden:

```js
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272")).json()).features[0];
const overlay = encodeURIComponent(JSON.stringify({ type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry }));
const img = `https://api.mapx.at/v1/static/satellite/geojson(${overlay})/auto/800x500.webp`;
```

## Nächste Schritte

- [Tilequery und Parcel API](/de/api/tilequery/)
- [Tilesets](/de/tilesets/cadastre/): Layer und Felder des Katasters
- [Höhen](/de/guides/elevation/): Höhe des Grundstücks
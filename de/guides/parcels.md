# Grundstücke und Kataster abfragen

Source: https://docs.mapx.at/de/guides/parcels/

> Das Katastergrundstück an einem Punkt finden, Grundstücksgrenzen auf der Karte zeichnen und die Attribute lesen: Kataster-Abfrage mit mapx für Österreich.

Der Kataster (BEV, DKM) steht als Vektor-Tileset zum Zeichnen und als API für das Grundstück an einem Punkt bereit. Die Daten dienen der Orientierung und sind nicht rechtsverbindlich.

| Aufgabe | Verwendung |
|---|---|
| Grundstücksgrenzen zeichnen | Tileset `cadastre` (z16) |
| Grundstücksnummer, Fläche und Katastralgemeinde an einem Punkt | [`/v1/parcel/{lon},{lat}`](https://docs.mapx.at/de/api/tilequery.md) |
| Ein beliebiges Vektor-Tileset an einem Punkt abfragen | [`/v1/tilequery/{tilesets}/{lon},{lat}.json`](https://docs.mapx.at/de/api/tilequery.md) |
| Ein Grundstück suchen | [Geocoding](https://docs.mapx.at/de/guides/search.md): Grundstücke gehören zu den durchsuchten Objekten |

## Grundstück an einem Punkt

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_number, cadastral_municipality_id, area_m2, is_building_land}'
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
  const res = await fetch(`https://api.mapx.at/v1/parcel/${lng},${lat}`, { headers: { Authorization: "Bearer DEIN_TOKEN" } });
  const [parcel] = (await res.json()).features;
  if (!parcel) return;
  const p = parcel.properties;
  new maplibregl.Popup().setLngLat(e.lngLat)
    .setHTML(`Grundstück ${p.parcel_number}<br>${Math.round(p.area_m2)} m²`).addTo(map);
});
```

Vollständiges Beispiel: [Grundstück per Klick](https://docs.mapx.at/de/examples/parcel-on-click.md). Die Katastralgemeinde (`cadastral_municipality_id`) gehört zur Identität eines Grundstücks: Die Nummer allein ist nicht eindeutig.

## Tilequery

Tilequery liest das Vektor-Tile im höchsten Zoom und liefert die Features, die dem Punkt am nächsten liegen (`radius` bis 2000 m).

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/tilequery/cadastre/15.3294,48.2272.json?radius=30&limit=3" | jq '.features[].properties.tilequery'
```

Tilesets lassen sich mit Komma kombinieren, zum Beispiel `streets,cadastre`. Jedes Feature hat `tilequery.distance` (m), `geometry` und `layer`.

## Grundstücksbild für ein Inserat

Das Grundstück mit einem [Static Image](https://docs.mapx.at/de/guides/static-images.md) kombinieren und das Polygon als GeoJSON-Overlay verwenden:

```js
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).features[0];
const overlay = encodeURIComponent(JSON.stringify({ type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry }));
const img = `https://api.mapx.at/v1/static/satellite/geojson(${overlay})/auto/800x500.webp?access_token=DEIN_TOKEN`;
```

## Grundstücks-Dossier

`GET /v1/parcels/{id}` liefert alles, was die offenen Daten zu einem Grundstück sagen, in einem Aufruf, für alle rund 10,2 Mio. Grundstücke Österreichs (DKM-Stichtag in `reference_date`). Die ID ist `<KG-Nummer>-<Grundstücksnummer>` mit `_` statt `/`, Bauparzellen mit Punkt (`14143-288_4`, `14143-.98`); sie steht als `properties.parcel_id` in Suchtreffern und der Punktabfrage.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/parcels/14143-288_4" | jq '.properties | {area_m2, land_use, buildings: .buildings.count, elevation, nearest_transit_stop}'
```

| Feld | Inhalt |
|---|---|
| `cadastral_municipality`, `municipality`, `district`, `state` | `{id, name}` |
| `area_m2`, `area_ellipsoidal_m2` | Fläche in der Gauß-Krüger-Ebene (so rechnet der Kataster) und auf dem Ellipsoid; der Registerwert gehört nicht zu den offenen Daten |
| `land_use` | DKM-Nutzungen mit `area_m2` und `share` |
| `buildings` | Anzahl, Grundfläche, Höhen aus BEV DLM wo vorhanden, die 25 größten |
| `elevation` | Min, Mittel, Max und mittlere/maximale Neigung aus dem Geländemodell an einem Punktraster im Grundstück |
| `addresses`, `nearest_transit_stop` | Adressen des Registers auf dem Grundstück, nächste Haltestelle im Umkreis von 2 km |
| `orthophoto` | Flugjahr und -datum |
| `climate` | Sonnenpotenzial, Temperatur- und Niederschlagsnormale, Hitze- und Schneetage, Dürreindex; dasselbe Objekt wie [`/v1/climate/{lon},{lat}`](https://docs.mapx.at/de/api/climate.md) am inneren Punkt des Grundstücks. Solange ein neues Gebiet geladen wird, sind Abschnitte `null` mit `status: "partial"`; nach `retry_after_s` Sekunden erneut fragen |
| `links.static_image` | fertige [Static-Images](https://docs.mapx.at/de/guides/static-images.md)-URL mit umrandetem Grundstück |

Die Punktabfrage ergänzt `parcel_id` und `links.dossier`; `?include=dossier` liefert je Feature das volle Dossier. Referenz: [Tilequery und Parcel API](https://docs.mapx.at/de/api/tilequery.md).

## Grundstückssuche

Geocoding und Search Box kennen `types=parcel` für alle Katastralgemeinden:

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=KG+14143+Gst+288/4&types=parcel"   # exakter Treffer
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=14143/288/4&types=parcel"
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk+.98&types=parcel"             # Bauparzelle
```

`KG 14143 Gst 288/4`, `14143/288/4` und `14143 288/4` sind eindeutig (eine fünfstellige Zahl neben einer Grundstücksnummer ist die KG-Nummer, Postleitzahlen haben vier Stellen). `Melk 288/4` stellt Treffer mit dieser Nummer nach vorn.

## Nächste Schritte

- [Tilequery und Parcel API](https://docs.mapx.at/de/api/tilequery.md)
- [Tilesets](https://docs.mapx.at/de/tilesets/cadastre.md): Layer und Felder des Katasters
- [Höhen](https://docs.mapx.at/de/guides/elevation.md): Höhe des Grundstücks

# Real estate

Parcel dossier, a static image for the listing and reachability of an address, with code that runs as it is.

For a listing you need three things: facts about the parcel, a picture that shows where it lies, and an answer to "how well can I reach it". All of it comes from official open data (BEV cadastre, orthophoto, GIP, timetables) without a key. The data is for orientation and not legally binding.

## Find the parcel

Search by parcel number (`KG 14105 Gst 102/3`, `14105/102/3`) or take the parcel at a point, for example from a click or an address:

```bash
curl -s "https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&types=address&limit=1" | jq '.features[0].center'
curl -s "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_id, parcel_number, area_m2, is_building_land}'
curl -s "https://api.mapx.at/v1/geocode?q=KG%2014105%20Gst%20102/3&types=parcel&limit=1" | jq '.features[0].properties.parcel_id'
```

## Read the dossier

`/v1/parcels/{id}` returns everything the open data say about one parcel in a single call: municipality and district, area, land use shares, buildings with heights where the BEV has them, terrain statistics, addresses, the nearest transit stop, the flight year of the orthophoto and a ready-made image link.

```js
const id = "14143-473_1"; // properties.parcel_id from the search or the point query
const d = await (await fetch(`https://api.mapx.at/v1/parcels/${id}`)).json();
const p = d.properties;
console.log(
  `${p.parcel_number} in ${p.cadastral_municipality.name}, ${Math.round(p.area_m2)} m²`,
  `| buildings: ${p.buildings.count}`,
  `| ${p.elevation?.min_m}-${p.elevation?.max_m} m, slope ${p.elevation?.slope_mean_deg}°`,
  `| nearest stop: ${p.nearest_transit_stop?.name} (${p.nearest_transit_stop?.distance_m} m)`,
);
```

Missing data are `null` or empty lists, never absent. With the [SDK](/guides/sdk/helpers/): `mapx.parcelDossier(id)` or `mapx.parcel(point, { dossier: true })`. For a map click use `new mapx.ParcelPopup(map)`.

## A picture for the listing

The dossier carries `links.static_image`: a satellite image with the parcel outline. To build your own, draw the polygon as a GeoJSON overlay and let the image fit it with `auto`:

```js
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272")).json()).features[0];
const outline = { type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry };
const img = `https://api.mapx.at/v1/static/satellite/geojson(${encodeURIComponent(JSON.stringify(outline))})/auto/800x500.webp`;
document.querySelector("#expose").src = img; // also fine in e-mails and PDFs
```

Use `outdoors` or `streets` instead of `satellite` for a plan view. Add `@2x` to the size for sharp images on high-resolution screens. See [Static images](/guides/static-images/). The image carries the attribution of its data; keep it visible in the layout ([Attribution](/reference/attribution/)).

## How well can I reach it

Show the area reachable on foot or by bike, and the distance to places that matter.

```js
const here = [15.3294, 48.2272];
const iso = await (await fetch(`https://api.mapx.at/v1/isochrone/walking/${here}?contours_minutes=5,10,15`)).json(); // GeoJSON polygons

const places = [here, [15.3333, 48.2279], [15.3201, 48.2218]]; // address, school, station
const m = await (await fetch(`https://api.mapx.at/v1/matrix/walking/${places.map((p) => p.join(",")).join(";")}?sources=0`)).json();
console.log(m.durations[0].map((s) => Math.round(s / 60) + " min"));
```

For public transport use the next stop from the dossier and its [departures](/guides/transit/): `GET /v1/departures?stop=<stop id>&limit=5`.

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=3" | jq '.departures[] | {time, line: .line.name, headsign}'
```

</TabItem>
<TabItem label="Python">

```python
r = requests.get("https://api.mapx.at/v1/departures", params={"stop": "at:49:1320", "limit": 3}).json()
print([(d["time"][11:16], d["line"]["name"]) for d in r["departures"]])
```

</TabItem>
</Tabs>

## Next steps

- [Parcels](/guides/parcels/) and the [Parcel on click](/examples/parcel-on-click/) example
- [Static images](/guides/static-images/) with markers and paths
- [Navigation](/guides/navigation/) for isochrones and matrices
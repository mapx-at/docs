# Real estate


For a listing you need three things: facts about the parcel, a picture that shows where it lies, and an answer to "how well can I reach it". All of it comes from official open data (BEV cadastre, orthophoto, GIP, timetables). You need a free API token ([Access and limits](/start/access-and-limits/)); the samples show it as `YOUR_TOKEN`. The data is for orientation and not legally binding.

## Find the parcel

Search by parcel number (`KG 14143 Gst 473/1`, `14143/473/1`) or take the parcel at a point, for example from a click or an address:

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Herrengasse%2013,%201010%20Wien&types=address&limit=1" | jq '.features[0].center'
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_id, parcel_number, area_m2, is_building_land}'
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=KG%2014143%20Gst%20473/1&types=parcel&limit=1" | jq '.features[0].properties.parcel_id'
```

## Read the dossier

`/v1/parcels/{id}` returns everything the open data say about one parcel in a single call: municipality and district, area, land use shares, buildings with heights where the BEV has them, terrain statistics, addresses, the nearest transit stop, the flight year of the orthophoto and a ready-made image link.

```js
const id = "14143-473_1"; // properties.parcel_id from the search or the point query
const d = await (await fetch(`https://api.mapx.at/v1/parcels/${id}?access_token=YOUR_TOKEN`)).json();
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
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272?access_token=YOUR_TOKEN")).json()).features[0];
const outline = { type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry };
const img = `https://api.mapx.at/v1/static/satellite/geojson(${encodeURIComponent(JSON.stringify(outline))})/auto/800x500.webp?access_token=YOUR_TOKEN`;
document.querySelector("#expose").src = img; // also fine in e-mails and PDFs
```

Use `outdoors` or `streets` instead of `satellite` for a plan view. Add `@2x` to the size for sharp images on high-resolution screens. See [Static images](/guides/static-images/). The image URL contains the token, so use a public token (`mx_pk_…`) with your site as allowed origin for images on web pages, and render images for e-mails and PDFs on your server with a secret token (`mx_sk_…`). The image carries the attribution of its data; keep it visible in the layout ([Attribution](/reference/attribution/)).

## Sun, heat and snow at the site

Buyers ask for solar yield, summer heat and snow load. `/v1/climate/{lon},{lat}` answers from GeoSphere Austria data; the same object is `properties.climate` in the dossier (taken at the inner point of the parcel). Each section names its dataset, grid cell and last day of data.

```js
const c = await (await fetch("https://api.mapx.at/v1/climate/15.3294,48.2272?access_token=YOUR_TOKEN")).json();
console.log(
  `Global radiation: ${c.solar?.global_radiation_kwh_m2_year} kWh/m² per year, ${c.solar?.sunshine_hours_year} h of sunshine`,
  `| mean temperature ${c.temperature.mean_annual_c.normal} °C (1991-2020), trend +${c.temperature.trend_c_per_decade} °C per decade`,
  `| hot days (TX ≥ 30 °C): ${c.temperature_extremes.hot_days_per_year.normal} → ${c.temperature_extremes.hot_days_per_year.recent} per year`,
  `| snow days: ${c.snow.snow_days_per_winter.normal} per winter, now ${c.snow.current.depth_cm} cm`,
  `| drought (SPEI 365): ${c.drought.spei365.class}`,
);
```

A rough photovoltaic estimate is the radiation on the roof times the module efficiency and a performance ratio (about 0.8); the value is for orientation and replaces no yield study. The first request for an area that nobody has asked for can take up to 20 seconds, later ones answer from a cache. In the dossier the section is `null` with `status: "partial"` until the data are loaded; ask again after `retry_after_s`. Reference: [Climate API](/api/climate/). Show the credit: GeoSphere Austria, CC BY 4.0 ([Attribution](/reference/attribution/)).

## How well can I reach it

Show the area reachable on foot or by bike, and the distance to places that matter.

```js
const here = [15.3294, 48.2272];
const iso = await (await fetch(`https://api.mapx.at/v1/isochrone/walking/${here}?contours_minutes=5,10,15&access_token=YOUR_TOKEN`)).json(); // GeoJSON polygons

const places = [here, [15.3333, 48.2279], [15.3201, 48.2218]]; // address, school, station
const m = await (await fetch(`https://api.mapx.at/v1/matrix/walking/${places.map((p) => p.join(",")).join(";")}?sources=0`)).json();
console.log(m.durations[0].map((s) => Math.round(s / 60) + " min"));
```

For public transport use the next stop from the dossier and its [departures](/guides/transit/): `GET /v1/departures?stop=<stop id>&limit=5`.

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=3" | jq '.departures[] | {time, line: .line.name, headsign}'
```

**Python**


```python
import requests
r = requests.get("https://api.mapx.at/v1/departures", params={"stop": "at:49:1320", "limit": 3},
                 headers={"Authorization": "Bearer YOUR_TOKEN"}).json()
print([(d["time"][11:16], d["line"]["name"]) for d in r["departures"]])
```


## Next steps

- [Parcels](/guides/parcels/) and the [Parcel on click](/examples/parcel-on-click/) example
- [Static images](/guides/static-images/) with markers and paths
- [Navigation](/guides/navigation/) for isochrones and matrices


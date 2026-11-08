# Parcels

Find the cadastral parcel at a point, draw parcel boundaries and read their attributes.

The cadastre (BEV, DKM) is available as a vector tileset for drawing and as an API for the parcel at a point. The data is for orientation and not legally binding. Every request needs a token ([Access and limits](/start/access-and-limits/)); `curl` samples send it as a header, browser samples as `access_token`.

| Task | Use |
|---|---|
| Draw parcel boundaries | tileset `cadastre` (z16) |
| Parcel number, area and cadastral municipality at a point | [`/v1/parcel/{lon},{lat}`](/api/tilequery/) |
| Query any vector tileset at a point | [`/v1/tilequery/{tilesets}/{lon},{lat}.json`](/api/tilequery/) |
| Search for a parcel | [Geocoding](/guides/search/): parcels are among the searched objects |

## Parcel at a point

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_number, cadastral_municipality_id, area_m2, is_building_land}'
```

The result is a `FeatureCollection` with the parcel polygon. The area is computed from the boundary lines and deviates from the register value by per mille. A building returns its building parcel; the number then starts with a dot. Parcels that extend beyond the 2 km tile window around the point are missing, and the list is empty outside the cadastre data.

## Click a parcel on the map

```js
map.on("load", () => {
  map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
  map.addLayer({ id: "parcel-lines", type: "line", source: "cadastre", "source-layer": "parcel_l", paint: { "line-color": "#e11d48" } });
});
map.on("click", async (e) => {
  const { lng, lat } = e.lngLat;
  const res = await fetch(`https://api.mapx.at/v1/parcel/${lng},${lat}?access_token=YOUR_TOKEN`);
  const [parcel] = (await res.json()).features;
  if (!parcel) return;
  const p = parcel.properties;
  new maplibregl.Popup().setLngLat(e.lngLat)
    .setHTML(`Parcel ${p.parcel_number}<br>${Math.round(p.area_m2)} m²`).addTo(map);
});
```

Full example: [Parcel on click](/examples/parcel-on-click/). The cadastral municipality (`cadastral_municipality_id`) is part of the identity of a parcel: the number alone is not unique.

## Tilequery

Tilequery reads the vector tile at the highest zoom and returns the features nearest to the point (`radius` up to 2000 m).

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tilequery/cadastre/15.3294,48.2272.json?radius=30&limit=3" | jq '.features[].properties.tilequery'
```

Combine tilesets with a comma, for example `streets,cadastre`. Each feature has `tilequery.distance` (m), `geometry` and `layer`.

## Parcel image for a listing

Combine the parcel with a [static image](/guides/static-images/) using the polygon as GeoJSON overlay:

```js
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272?access_token=YOUR_TOKEN")).json()).features[0];
const overlay = encodeURIComponent(JSON.stringify({ type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry }));
const img = `https://api.mapx.at/v1/static/satellite/geojson(${overlay})/auto/800x500.webp?access_token=YOUR_TOKEN`;
```

## Parcel dossier

`GET /v1/parcels/{id}` returns everything the open data say about one parcel in one call, for all roughly 10.2 million parcels of Austria (DKM key date in `reference_date`). The ID is `<KG number>-<parcel number>` with `/` written as `_`, building parcels with a leading dot (`14143-473_1`, `14143-.98`); it is `properties.parcel_id` in search results and in the point query.

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcels/14143-473_1" | jq '.properties | {area_m2, land_use, buildings: .buildings.count, elevation, nearest_transit_stop}'
```

| Field | Content |
|---|---|
| `cadastral_municipality`, `municipality`, `district`, `state` | `{id, name}` |
| `area_m2`, `area_ellipsoidal_m2` | area in the Gauss-Krueger plane (as the cadastre computes it) and on the ellipsoid; the register value is not part of the open data |
| `land_use` | DKM land use classes with `area_m2` and `share` |
| `buildings` | count, footprint, heights from BEV DLM where available, the 25 largest |
| `elevation` | min, mean, max and mean/max slope from the terrain model on a point grid inside the parcel |
| `addresses`, `nearest_transit_stop` | register addresses on the parcel, nearest stop within 2 km |
| `orthophoto` | flight year and date |
| `climate` | solar potential, temperature and precipitation normals, hot and snow days, drought index; same object as [`/v1/climate/{lon},{lat}`](/api/climate/) at the parcel's inner point. While a new area is being loaded, sections are `null` with `status: "partial"`; ask again after `retry_after_s` seconds |
| `links.static_image` | ready-made [static image](/guides/static-images/) URL with the parcel outlined |

The point query adds `parcel_id` and `links.dossier`; `?include=dossier` returns the full dossier per feature. Reference: [Tilequery and Parcel API](/api/tilequery/).

## Parcel search

Geocoding and Search Box know `types=parcel` for all cadastral municipalities:

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=KG+14143+Gst+288/4&types=parcel"   # exact hit
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=14143/288/4&types=parcel"
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk+.98&types=parcel"             # building parcel
```

`KG 14143 Gst 288/4`, `14143/288/4` and `14143 288/4` are unambiguous (a five-digit number next to a parcel number is the KG number, postcodes have four digits). `Melk 288/4` puts hits with that number first.

## Next

- [Tilequery and Parcel API](/api/tilequery/)
- [Tilesets](/tilesets/cadastre/): layers and fields of the cadastre
- [Elevation](/guides/elevation/): height of the parcel
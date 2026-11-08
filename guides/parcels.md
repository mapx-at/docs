# Parcels

Find the cadastral parcel at a point, draw parcel boundaries and read their attributes.

The cadastre (BEV, DKM) is available as a vector tileset for drawing and as an API for the parcel at a point. The data is for orientation and not legally binding.

| Task | Use |
|---|---|
| Draw parcel boundaries | tileset `cadastre` (z16) |
| Parcel number, area and cadastral municipality at a point | [`/v1/parcel/{lon},{lat}`](/api/tilequery/) |
| Query any vector tileset at a point | [`/v1/tilequery/{tilesets}/{lon},{lat}.json`](/api/tilequery/) |
| Search for a parcel | [Geocoding](/guides/search/): parcels are among the searched objects |

## Parcel at a point

```bash
curl -s "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq '.features[0].properties | {parcel_number, cadastral_municipality_id, area_m2, is_building_land}'
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
  const res = await fetch(`https://api.mapx.at/v1/parcel/${lng},${lat}`);
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
curl -s "https://api.mapx.at/v1/tilequery/cadastre/15.3294,48.2272.json?radius=30&limit=3" | jq '.features[].properties.tilequery'
```

Combine tilesets with a comma, for example `streets,cadastre`. Each feature has `tilequery.distance` (m), `geometry` and `layer`.

## Parcel image for a listing

Combine the parcel with a [static image](/guides/static-images/) using the polygon as GeoJSON overlay:

```js
const parcel = (await (await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272")).json()).features[0];
const overlay = encodeURIComponent(JSON.stringify({ type: "Feature", properties: { stroke: "#e11d48", "stroke-width": 3, fill: "#e11d48", "fill-opacity": 0.2 }, geometry: parcel.geometry }));
const img = `https://api.mapx.at/v1/static/satellite/geojson(${overlay})/auto/800x500.webp`;
```

## Next

- [Tilequery and Parcel API](/api/tilequery/)
- [Tilesets](/tilesets/cadastre/): layers and fields of the cadastre
- [Elevation](/guides/elevation/): height of the parcel
# Elevation guide: height profile and terrain

Source: https://docs.mapx.at/guides/elevation/

> Get heights at points and elevation profiles, use terrain models and show 3D terrain in the map: elevation API and terrain tilesets for Austria.

mapx has two height models of 1 m resolution (BEV, laser scanning): the terrain model `dtm` (bare ground) and the surface model `dsm` (including trees and buildings). Use `dtm` for gradients, profiles and trails, `dsm` for visibility and building heights. Every request needs a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); the samples send it in the `Authorization` header, which browsers may send too.

## Height at a point

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation/12.694,47.074" | jq '.properties | {dtm, dsm}'
```

Several points, up to 512, in one request:

```js
const url = "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
console.log(features.map((f) => f.properties.elevation)); // meters above sea level
```

Heights are in meters with 0.1 m steps. Outside the data area the value is `null`.

## Elevation profile

```js
const url = "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=50";
const { properties: p } = await (await fetch(url, { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json();
console.log(p.ascent_m, p.descent_m, p.min_m, p.max_m);
// p.distances[i] (m from the start) and p.elevations[i] (m) are ready for a chart
```

Up to 1024 samples along a line of 2 to 512 vertices. Ascent and descent are counted with a 3 m hysteresis (`ascent_threshold_m`): a rise or fall counts only once the elevation has moved at least 3 m from the last turning point, so terrain noise does not add up and the totals hardly depend on `samples`. With `model=dsm` trees and buildings count as terrain and the ascent comes out higher.

A route geometry goes straight into the profile with `POST /v1/elevation/profile` (GeoJSON LineString, a Feature or `{"coordinates": [...]}`, up to 20000 vertices):

```js
const route = (await (await fetch("https://api.mapx.at/v1/directions/cycling/12.8418,47.0378;12.7303,47.0792?geometries=geojson&overview=full", { headers: { Authorization: "Bearer YOUR_TOKEN" } })).json()).routes[0];
const prof = await (await fetch("https://api.mapx.at/v1/elevation/profile?samples=200", {
  method: "POST",
  headers: { Authorization: "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify(route.geometry),
})).json();
console.log(prof.properties.ascent_m);
```

 Example with a chart: [Elevation profile](https://docs.mapx.at/examples/elevation-profile.md). Routes already carry ascent and descent in `routes[].summary`; use `annotations=elevation` for a height per segment.

## 3D terrain in the map

Add the `terrain-rgb` tileset as `raster-dem` source and turn on terrain:

```js
map.on("load", () => {
  // two separate sources: one for the terrain, one for the hillshade (better rendering)
  map.addSource("dem", { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512 });
  map.addSource("dem-hillshade", { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512 });
  map.setTerrain({ source: "dem", exaggeration: 1.2 });
  map.addLayer({ id: "hillshade", type: "hillshade", source: "dem-hillshade" });
  map.setPitch(60);
});
```

Step-by-step with aerial imagery: [3D terrain with orthophoto](https://docs.mapx.at/guides/3d-terrain.md). The style `outdoors` already includes hillshade and contours; the ready-made 3D styles `outdoors-3d` and `satellite-3d` bring the terrain along, see [Maps](https://docs.mapx.at/guides/maps.md).

## Read heights from tiles

The tiles use the Terrain-RGB encoding: `height = -10000 + (R * 65536 + G * 256 + B) * 0.1`. Decode tiles yourself for bulk work, see [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md). For single points the API is simpler.

## Next

- [Elevation API](https://docs.mapx.at/api/elevation.md)
- [Tilesets](https://docs.mapx.at/tilesets/terrain-rgb.md): terrain model data sheet
- [Overlay contour lines](https://docs.mapx.at/guides/overlay-contours.md)

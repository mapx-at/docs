# Elevation and terrain

mapx has two height models of 1 m resolution (BEV, laser scanning): the terrain model `dtm` (bare ground) and the surface model `dsm` (including trees and buildings). Use `dtm` for gradients, profiles and trails, `dsm` for visibility and building heights. Every request needs a token ([Access and limits](/start/access-and-limits/)); `curl` samples send it as a header, browser samples as `access_token`.

## Height at a point

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/elevation/12.694,47.074" | jq '.properties | {dtm, dsm}'
```

Several points, up to 512, in one request:

```js
const url = "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm&access_token=YOUR_TOKEN";
const { features } = await (await fetch(url)).json();
console.log(features.map((f) => f.properties.elevation)); // meters above sea level
```

Heights are in meters with 0.1 m steps. Outside the data area the value is `null`.

## Elevation profile

```js
const url = "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=50&access_token=YOUR_TOKEN";
const { properties: p } = await (await fetch(url)).json();
console.log(p.ascent_m, p.descent_m, p.min_m, p.max_m);
// p.distances[i] (m from the start) and p.elevations[i] (m) are ready for a chart
```

Up to 1024 samples along a line of 2 to 512 vertices. Example with a chart: [Elevation profile](/examples/elevation-profile/). Routes already carry ascent and descent in `routes[].summary`; use `annotations=elevation` for a height per segment.

## 3D terrain in the map

Add the `terrain-rgb` tileset as `raster-dem` source and turn on terrain:

```js
map.on("load", () => {
  map.addSource("dem", { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512 });
  map.setTerrain({ source: "dem", exaggeration: 1.2 });
  map.addLayer({ id: "hillshade", type: "hillshade", source: "dem" });
});
map.setPitch(60);
```

Step-by-step with aerial imagery: [3D terrain with orthophoto](/guides/3d-terrain/). The style `outdoors` already includes hillshade and contours.

## Read heights from tiles

The tiles use the Terrain-RGB encoding: `height = -10000 + (R * 65536 + G * 256 + B) * 0.1`. Decode tiles yourself for bulk work, see [Terrain-RGB](/reference/terrain-rgb/). For single points the API is simpler.

## Next

- [Elevation API](/api/elevation/)
- [Tilesets](/tilesets/terrain-rgb/): terrain model data sheet
- [Overlay contour lines](/guides/overlay-contours/)


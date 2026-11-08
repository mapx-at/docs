# Attribution and license


All mapx data is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.en). You may use it commercially. You must name the source and the license and must not present modifications as official. The credit has to be visible to the users of the map, for example in the corner, not only in an imprint.

## In a MapLibre map

MapLibre shows the attribution automatically when you load a style or a TileJSON (bottom right, "i" icon). You add nothing as long as you do not hide it. If you set `attributionControl: false` or use `tiles: [...]` without TileJSON, add the text yourself:

```js
{ type: "raster", tiles: ["https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN"], tileSize: 512,
  maxzoom: 18, attribution: "Orthophoto: BEV (CC BY 4.0)" }
```

## In print, reports and static images

Put a visible line next to the image, in readable size, for example `Source: BEV – ALS-DTM 1 m (2025-09-15), CC BY 4.0`. With several sources list them all, separated by semicolons.

## Text per tileset

| Tileset | Text |
|---|---|
| `streets` | {{< ts k="basis" f="attribution" >}} |
| `cadastre` | {{< ts k="kataster" f="attribution" >}} |
| `contours` | {{< ts k="hoehenlinien" f="attribution" >}} |
| `terrain-rgb` | {{< ts k="relief" f="attribution" >}} |
| `dsm` | {{< ts k="dsm" f="attribution" >}} |
| `orthophoto` | {{< ts k="dop" f="attribution" >}} |
| `orthophoto-flight-blocks` | BEV, CC BY 4.0 |

## Text per API

JSON responses with data carry a top-level `attribution` field. Show it when you display or pass on the data.

| API | Source |
|---|---|
| Geocoding, Search Box, Address | BEV (address register, geographical names, administrative boundaries, cadastre), CC BY 4.0 |
| Directions, Isochrone, Matrix, Matching | GIP.at, CC BY 4.0 |
| Tolls, Traffic | ASFINAG / EVIS.AT, CC BY 4.0, road network GIP.at |
| Transit | ÖBB-Personenverkehr AG (CC BY 4.0), data source Mobilitätsverbünde Österreich OG |
| Weather | GeoSphere Austria, CC BY 4.0 |
| Climate, `properties.climate` of the parcel dossier | GeoSphere Austria (APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE), CC BY 4.0 |
| Parcel, Tilequery (cadastre) | BEV – DKM, CC BY 4.0 |
| Elevation | BEV – ALS-DTM / ALS-DOM 1 m, CC BY 4.0 |

The credit is also stored in the TileJSON and PMTiles metadata.


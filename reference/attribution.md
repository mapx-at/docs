# Attribution and license (CC BY 4.0)

Source: https://docs.mapx.at/reference/attribution/

> Which source to credit under CC BY 4.0 when you show mapx maps or data: ready-made attribution text per tileset and API, for MapLibre, print and images.

The official data in mapx is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.en). The Sentinel-2 tiles are the exception: they contain modified Copernicus Sentinel data and follow the Copernicus terms; the credit is in the TileJSON. You may use it commercially. You must name the source and the license and must not present modifications as official. The credit has to be visible to the users of the map, for example in the corner, not only in an imprint.

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
| `streets` | Datenquellen: BEV (DKM, Verwaltungsgrenzen, Adressregister, Geographische Namen), GIP.at, BMLUK (GGN), Umweltbundesamt; CC BY 4.0 |
| `cadastre` | Datenquelle: BEV – DKM, CC BY 4.0 |
| `contours` | Höhenlinien: mapx.at, Datenquelle BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `terrain-rgb` | Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `dsm` | Relief: BEV – ALS-DOM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `orthophoto` | Orthophoto: BEV (CC BY 4.0) |
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

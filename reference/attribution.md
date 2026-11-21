# Attribution and license (CC BY 4.0)

Source: https://docs.mapx.at/reference/attribution/

> Which source to credit under CC BY 4.0 when you show mapx maps or data: ready-made attribution text per tileset and API, for MapLibre, print and images.

The official data in mapx is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.en). The Sentinel-2 tiles are the exception: they contain modified Copernicus Sentinel data and follow the Copernicus terms; the credit is in the TileJSON. You may use it commercially. You must name the source and the license and must not present modifications as official. Users must be able to reach the credit on the map itself, for example as text in the corner or behind an "i" info button on the map, not only in an imprint.

## mapx.at first

Every mapx attribution starts with `mapx.at`, followed by the data sources. The styles under `/v1/styles` carry `mapx.at` with a link to https://mapx.at as their first entry (own source `mapx` with the layer `mapx-attribution`); please do not remove it. If you add tilesets directly you get the link through the TileJSON (`attribution` starts with `<a href="https://mapx.at" …>mapx.at</a>`), the JSON responses of the API through their `attribution` field (`mapx.at; …`). MapLibre orders the entries by length, shortest first: your own sources with a very short attribution (under 68 characters) appear before it. With Leaflet, OpenLayers and static images put `mapx.at` in front yourself, as in the examples.

## In a MapLibre map

MapLibre shows the attribution automatically when you load a style or a TileJSON (bottom right). The examples in these docs collapse it so it does not cover the map: `attributionControl: false` in the map options and `map.addControl(new maplibregl.AttributionControl({ compact: true }))` show an "i" button, a click shows the sources. MapLibre first shows the compact form open and collapses it on the first map drag; the `map.once("load", …)` line from the examples collapses it right after loading. `attributionControl: false` without your own `AttributionControl` hides the sources completely, which is not allowed. If you use `tiles: [...]` without TileJSON, add the text to the source yourself:

```js
{ type: "raster", tiles: ["https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN"], tileSize: 512,
  maxzoom: 18, attribution: "mapx.at; Orthophoto: BEV, CC BY 4.0" }
```

## In print, reports and static images

Put a visible line next to the image, in readable size, for example `Source: mapx.at; BEV – ALS-DTM 1 m (2025-09-15), CC BY 4.0`. With several sources list them all, separated by semicolons.

## Text per tileset

| Tileset | Text |
|---|---|
| `streets` | mapx.at; Datenquellen: BEV (DKM, Verwaltungsgrenzen, Adressregister, Geographische Namen), GIP.at, BMLUK (GGN), Umweltbundesamt, Stadt Wien – data.wien.gv.at; CC BY 4.0 |
| `contours` | mapx.at; Höhenlinien: mapx.at, Datenquelle BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `terrain-rgb` | mapx.at; Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `dsm` | mapx.at; Relief: BEV – ALS-DOM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `orthophoto` | mapx.at; Orthophoto: BEV (CC BY 4.0) |
| `orthophoto-flight-blocks` | The TileJSON only names `mapx.at`; credit `mapx.at; BEV, CC BY 4.0`, since the flight blocks are metadata of the BEV orthophoto. |
| `bike-routes` | mapx.at; Datenquelle: GIP.at, CC BY 4.0 |
| `gip-poi` | mapx.at; Datenquelle: GIP.at, CC BY 4.0 |
| `rest-areas` | mapx.at; ASFINAG, CC BY 4.0 |
| `sharing` | mapx.at; ÖBB-Personenverkehr AG – Shared-Mobility Standorte, CC BY 4.0 |
| `sentinel2`, `sentinel2-date`, `sentinel2-ndvi`, `sentinel2-ndwi`, `sentinel2-ndsi` | mapx.at; Contains modified Copernicus Sentinel data 2026 (year of the images; stated in the TileJSON) |

## Text per API

JSON responses with data carry a top-level `attribution` field; it always starts with `mapx.at; ` before the source from the table. Show it when you display or pass on the data.

| API | Source |
|---|---|
| Geocoding, Search Box, Address | BEV (address register, geographical names, administrative boundaries), CC BY 4.0 |
| Directions, Isochrone, Matrix, Matching | GIP.at, CC BY 4.0 |
| Tolls, Traffic | ASFINAG / EVIS.AT, CC BY 4.0, road network GIP.at |
| Public transport | ÖBB-Personenverkehr AG (CC BY 4.0), data source Mobilitätsverbünde Österreich OG |
| Weather | GeoSphere Austria, CC BY 4.0 |
| Climate | GeoSphere Austria (APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE), CC BY 4.0 |
| Air quality | GeoSphere Austria (chemical weather model), CC BY 4.0 |
| Tilequery | Source of the queried tileset, see [Tilesets](https://docs.mapx.at/tilesets/index.md) |
| Elevation | BEV – ALS-DTM / ALS-DOM 1 m, CC BY 4.0 |

The credit is also stored in the TileJSON.

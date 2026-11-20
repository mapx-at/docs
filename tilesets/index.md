# Tilesets: vector and raster tiles

Source: https://docs.mapx.at/tilesets/

> All mapx tilesets for Austria with type, zoom range and source date: basemap, cadastre, contours, orthophoto, terrain and surface models.

Each tileset is available as TileJSON and XYZ tiles through the [Tiles API](https://docs.mapx.at/api/tiles.md) and is part of the [styles](https://docs.mapx.at/api/styles.md). All data is licensed CC BY 4.0, see [Attribution](https://docs.mapx.at/reference/attribution.md).

| Tileset | Type | Zoom | As of |
|---|---|---|---|
| [`streets`](https://docs.mapx.at/tilesets/streets.md) | vector | z0–16 | 2026-04-01 |
| [`cadastre`](https://docs.mapx.at/tilesets/cadastre.md) | vector | z16–16 | 2026-04-01 |
| [`contours`](https://docs.mapx.at/tilesets/contours.md) | vector | z10–16 | 20250915 |
| [`terrain-rgb`](https://docs.mapx.at/tilesets/terrain-rgb.md) | DEM | z5–16 | 20250915 |
| [`dsm`](https://docs.mapx.at/tilesets/dsm.md) | DEM | z5–16 | 20250915 |
| [`orthophoto`](https://docs.mapx.at/tilesets/orthophoto.md) | raster | z5–18 | 2025-10-03 |
| [`orthophoto-flight-blocks`](https://docs.mapx.at/tilesets/orthophoto-flight-blocks.md) | vector | z6–17 | 2025-10-03 |
| [`rest-areas`](https://docs.mapx.at/tilesets/rest-areas.md) | vector | z7–14 | 20261120 |
| [`sharing`](https://docs.mapx.at/tilesets/sharing.md) | vector | z8–14 | 20261120 |
| [`bike-routes`](https://docs.mapx.at/tilesets/bike-routes.md) | vector | z5–14 | 20260708 |
| [`gip-poi`](https://docs.mapx.at/tilesets/gip-poi.md) | vector | z12–15 | 20260708 |
| [`sentinel2`](https://docs.mapx.at/tilesets/sentinel2.md) | raster | z5–z13 | daily |

Not in the table because it is replaced daily instead of versioned: [Sentinel-2 current](https://docs.mapx.at/tilesets/sentinel2.md) (`sentinel2`, raster, z5 to z13).

Use a tileset by name:

```js
map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
```

The TileJSON address is `https://tiles.mapx.at/v1/tiles/{tileset}.json`.

## Next

- [Maps guide](https://docs.mapx.at/guides/maps.md)
- [Styles API](https://docs.mapx.at/api/styles.md)

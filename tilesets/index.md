# Tilesets: vector and raster tiles

Source: https://docs.mapx.at/tilesets/

> All mapx tilesets for Austria with type, zoom range and source date: basemap, contours, orthophoto, terrain and surface models.

Each tileset is available as TileJSON and XYZ tiles through the [Tiles API](https://docs.mapx.at/api/tiles.md) and is part of the [styles](https://docs.mapx.at/api/styles.md). The data is licensed CC BY 4.0 (exception: Sentinel-2), see [Attribution and license](https://docs.mapx.at/reference/attribution.md).

| Tileset | Type | Zoom | As of |
|---|---|---|---|
| [`streets`](https://docs.mapx.at/tilesets/streets.md) | vector | z0–16 | 2026-04-01 |
| [`contours`](https://docs.mapx.at/tilesets/contours.md) | vector | z10–16 | 2025-09-15 |
| [`terrain-rgb`](https://docs.mapx.at/tilesets/terrain-rgb.md) | DEM | z5–16 | 2025-09-15 |
| [`dsm`](https://docs.mapx.at/tilesets/dsm.md) | DEM | z5–16 | 2025-09-15 |
| [`orthophoto`](https://docs.mapx.at/tilesets/orthophoto.md) | raster | z5–18 | 2025-10-03 |
| [`orthophoto-flight-blocks`](https://docs.mapx.at/tilesets/orthophoto-flight-blocks.md) | vector | z6–17 | 2025-10-03 |
| [`rest-areas`](https://docs.mapx.at/tilesets/rest-areas.md) | vector | z7–14 | 2026-11-20 |
| [`sharing`](https://docs.mapx.at/tilesets/sharing.md) | vector | z8–14 | 2026-11-20 |
| [`bike-routes`](https://docs.mapx.at/tilesets/bike-routes.md) | vector | z5–14 | 2026-07-08 |
| [`gip-poi`](https://docs.mapx.at/tilesets/gip-poi.md) | vector | z12–15 | 2026-07-08 |
| [`sentinel2`](https://docs.mapx.at/tilesets/sentinel2.md) | raster | z5–z13 | daily |
| [`sentinel2-date`](https://docs.mapx.at/tilesets/sentinel2.md) | raster | z5–z13 | daily |
| [`sentinel2-ndvi`](https://docs.mapx.at/tilesets/sentinel2.md) | raster | z5–z13 | daily |
| [`sentinel2-ndwi`](https://docs.mapx.at/tilesets/sentinel2.md) | raster | z5–z13 | daily |
| [`sentinel2-ndsi`](https://docs.mapx.at/tilesets/sentinel2.md) | raster | z5–z13 | daily |

The Sentinel-2 tilesets ([Sentinel-2 current](https://docs.mapx.at/tilesets/sentinel2.md) and four derived products) are replaced daily instead of versioned, so they show "daily" instead of a reference date.

Use a tileset by name:

```js
map.addSource("contours", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" });
```

All requests to `tiles.mapx.at` need a token, see [Access and limits](https://docs.mapx.at/start/access-and-limits.md). MapLibre sources send it via `transformRequest`, tile URLs take `?access_token=YOUR_TOKEN`.

The TileJSON address is `https://tiles.mapx.at/v1/tiles/{tileset}.json`.

## Next

- [Maps guide](https://docs.mapx.at/guides/maps.md)
- [Styles API](https://docs.mapx.at/api/styles.md)

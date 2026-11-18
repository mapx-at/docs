# mapx data in QGIS: XYZ and vector tiles

Source: https://docs.mapx.at/guides/qgis/

> Add DTM, DSM, orthophoto and vector tiles to QGIS: XYZ connection and vector tile connection.

**Goal:** Use mapx orthophoto, elevation models and vector data as layers in QGIS, without a download. You need a free token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); append `?access_token=YOUR_TOKEN` to every layer URL. Use a secret token (`mx_sk_…`) on your own computer: a public token only works for its allowed origins, and QGIS is not a website.

**Prerequisites:** QGIS 3.x.

## Orthophoto as an XYZ layer

1. In the Browser panel, right-click **XYZ Tiles**, **New Connection**.
2. Name `mapx Orthophoto`, URL:

   ```
   https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN
   ```

3. **Min. Zoom Level** 5, **Max. Zoom Level** 18, **Tile Resolution** 512 (Mapbox style, 512 px). Leave **Referer** and **Authentication** empty; the token in the URL is enough.
4. Save with OK and load the connection into the map by double-click.

The project CRS can be anything, QGIS reprojects the Web Mercator tiles.

## DTM and DSM

The elevation tiles are [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md): the elevation is stored in the colour channels. Use the same kind of connection with the URL

```
https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN
```

(max. zoom 16; use `dsm` instead of `terrain-rgb` for the surface model). The layer first shows the encoded colour. Get the elevation in the **Raster Calculator** from the three bands:

```
-10000 + ("Layer@1" * 65536 + "Layer@2" * 256 + "Layer@3") * 0.1
```

(replace `Layer` with the name of your layer). The result is the elevation in metres; from it you can derive slope, aspect and hillshade.

## Vector tiles

1. Browser panel, right-click **Vector Tiles**, **New Generic Connection**.
2. Name `mapx Basemap`, URL:

   ```
   https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN
   ```

3. Enter **Min. Zoom Level** 0, **Max. Zoom Level** 16, OK.
4. Drag the connection into the map. QGIS first shows a default style; change the look in the layer style editor or with your own style.

The same procedure works for `contours` (max 16, min 10), `cadastre` (zoom 16) and `orthophoto-flight-blocks` (zoom 6 to 17). All tilesets: [Tiles](https://docs.mapx.at/api/tiles.md).

## Next steps

- Understand the elevation model: [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md).
- Credits in maps and reports: [Attribution](https://docs.mapx.at/reference/attribution.md).

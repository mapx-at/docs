# mapx data in QGIS


**Goal:** Use mapx orthophoto, elevation models and vector data as layers in QGIS, without a download. You need a free token ([Access and limits](/start/access-and-limits/)); append `?access_token=YOUR_TOKEN` to every layer URL. Use a secret token (`mx_sk_…`) on your own computer: a public token only works for its allowed origins, and QGIS is not a website.

**Prerequisites:** QGIS 3.x (3.32 or newer for direct PMTiles).

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

The elevation tiles are [Terrain-RGB](/reference/terrain-rgb/): the elevation is stored in the colour channels. Use the same kind of connection with the URL

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


The same procedure works for `contours` (max 16, min 10), `cadastre` (zoom 16) and `orthophoto-flight-blocks` (zoom 6 to 17). All tilesets: [Tiles](/api/tiles/).

## PMTiles directly (QGIS 3.32 and newer)

Vector tilesets can also be opened directly as a PMTiles file from QGIS 3.32, locally or over HTTP. The file name with its cut-off date is on each [product page](/tilesets/) under key figures.


1. Browser panel, right-click **Vector Tiles**, **New Generic Connection**.
2. Set the **Service type** to PMTiles (or pick a local file) and enter the file as URL: `https://tiles.mapx.at/<file>.pmtiles?access_token=YOUR_TOKEN`.
3. Save with OK and load it into the map. QGIS reads only the required tiles via HTTP range.


Imagery and elevation tiles (orthophoto, DTM, DSM) are added through the XYZ connection above.

> If your QGIS version does not offer the PMTiles service type, use the vector tile connection above; it delivers the same tiles.


## Next steps

- Understand the elevation model: [Terrain-RGB](/reference/terrain-rgb/).
- Keep data locally: [Self-host PMTiles](/guides/self-host-pmtiles/).
- Credits in maps and reports: [Attribution](/reference/attribution/).


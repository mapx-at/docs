# mapx data in QGIS

Add DTM, DSM, orthophoto and vector tiles to QGIS: XYZ connection, vector tile connection and PMTiles directly.

**Goal:** Use mapx orthophoto, elevation models and vector data as layers in QGIS, without a download and without a key.

**Prerequisites:** QGIS 3.x (3.32 or newer for direct PMTiles).

## Orthophoto as an XYZ layer

<Steps>

1. In the Browser panel, right-click **XYZ Tiles**, **New Connection**.
2. Name `mapx Orthophoto`, URL:

   ```
   https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp
   ```

3. **Min. Zoom Level** 5, **Max. Zoom Level** 18, **Tile Resolution** 512 (Mapbox style, 512 px). Leave **Referer** and **Authentication** empty.
4. Save with OK and load the connection into the map by double-click.

</Steps>

The project CRS can be anything, QGIS reprojects the Web Mercator tiles.

## DTM and DSM

The elevation tiles are [Terrain-RGB](/en/konzepte/terrain-rgb/): the elevation is stored in the colour channels. Use the same kind of connection with the URL

```
https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp
```

(max. zoom 16; use `dsm` instead of `dtm` for the surface model). The layer first shows the encoded colour. Get the elevation in the **Raster Calculator** from the three bands:

```
-10000 + ("Layer@1" * 65536 + "Layer@2" * 256 + "Layer@3") * 0.1
```

(replace `Layer` with the name of your layer). The result is the elevation in metres; from it you can derive slope, aspect and hillshade.

## Vector tiles

<Steps>

1. Browser panel, right-click **Vector Tiles**, **New Generic Connection**.
2. Name `mapx Basemap`, URL:

   ```
   https://tiles.mapx.at/v1/mapx-basis/{z}/{x}/{y}.pbf
   ```

3. Enter **Min. Zoom Level** 0, **Max. Zoom Level** 16, OK.
4. Drag the connection into the map. QGIS first shows a default style; change the look in the layer style editor or with your own style.

</Steps>

The same procedure works for `hoehenlinien` (max 16, min 10), `mapx-kataster` (zoom 16) and `dop-flugblock` (zoom 6 to 17). All tilesets: [Tiles](/en/api/kacheln/).

## PMTiles directly (QGIS 3.32 and newer)

Vector tilesets can also be opened directly as a PMTiles file from QGIS 3.32, locally or over HTTP. The file name with its cut-off date is on each [product page](/en/daten/) under key figures.

<Steps>

1. Browser panel, right-click **Vector Tiles**, **New Generic Connection**.
2. Set the **Service type** to PMTiles (or pick a local file) and enter the file as URL: `https://tiles.mapx.at/<file>.pmtiles`.
3. Save with OK and load it into the map. QGIS reads only the required tiles via HTTP range.

</Steps>

Imagery and elevation tiles (orthophoto, DTM, DSM) are added through the XYZ connection above.

> If your QGIS version does not offer the PMTiles service type, use the vector tile connection above; it delivers the same tiles.

## Next steps

- Understand the elevation model: [Terrain-RGB](/en/konzepte/terrain-rgb/).
- Keep data locally: [Self-host PMTiles](/en/anleitungen/pmtiles-selbst-hosten/).
- Credits in maps and reports: [Attribution](/en/anleitungen/attribution/).
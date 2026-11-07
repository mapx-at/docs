# Frequently asked questions

Answers to common questions on keys, licence, commercial use, freshness, offline use, accuracy and errors.

## Do I need an API key or an account?

No. Tiles, TileJSON and the elevation API work without a key and without sign-up; CORS is open. See [Quick start](/en/start/).

## What does it cost, and under which licence is the data?

Use is free. The data is under [CC BY 4.0](/en/lizenz/): you must name the source, everything else is free. [Attribution](/en/anleitungen/attribution/) shows how the credit looks.

## May I use the data commercially?

Yes. CC BY 4.0 allows commercial use, also in your own products and apps, as long as the source is credited.

## How current is the data?

Each product has a cut-off date of the official source, for example 15.09.2025 for elevation models and contour lines, October 2025 for the orthophoto, 01.04.2026 for basemap and cadastre. The date is on the [product page](/en/daten/), in the tile metadata and in the elevation API as `stichtag`. File names contain it.

## Does a URL change when there is new data?

PMTiles file names contain the cut-off date; a new release gets a new name, old files stay valid. To always get the current release, use the [TileJSON](/en/api/tilejson/) address `https://tiles.mapx.at/v1/{tileset}.json`.

## Can I use the map offline?

Yes. PMTiles are single files; you download a file or a subset and serve it yourself or read it locally. See [Self-host PMTiles](/en/anleitungen/pmtiles-selbst-hosten/) and [mapx in QGIS](/en/anleitungen/qgis/).

## Is there data outside Austria?

The products cover Austria. The orthophoto and basemap extend into neighbouring countries at the borders; the exact coverage is under "Key figures" (area) on the product pages and in `bounds` of the TileJSON. Outside it, tiles return `404`.

## How accurate are the elevations?

Accuracy matches the official BEV source (laser scanning, 1 m raster). The tiles store elevations in steps of 0.1 m. Details: [DTM terrain model](/en/daten/dtm/) and [Terrain-RGB](/en/konzepte/terrain-rgb/).

## What is the difference between DTM and DSM?

The **[DTM](/en/daten/dtm/)** (terrain model) shows the ground without vegetation and buildings. The **[DSM](/en/daten/dsm/)** (surface model) shows the surface including trees and structures. Use the DTM for trails, gradients and profiles, the DSM for visibility, solar planning and building heights. In the [elevation API](/en/api/hoehe/) pick with `model=dtm` or `model=dsm`.

## Why do I get 404 for a tile?

Typical reasons: the tile is outside the data area or zoom range, the file extension does not match the type (`webp` for raster, `pbf` for vector) or the tileset name is wrong. For raster tiles use `minzoom`/`maxzoom` from the TileJSON. See [Tiles](/en/api/kacheln/) and [Errors and limits](/en/api/fehler/).

## Why do I get 429?

The service limits requests per IP (default 50 per second). Wait the time given in the `Retry-After` header and retry with increasing delay. For higher throughput, host the files yourself, see [Self-host PMTiles](/en/anleitungen/pmtiles-selbst-hosten/). Details under [Errors and limits](/en/api/fehler/).

## My map is off by one zoom level or blurry. What now?

The raster tiles are 512 pixels. Set the tile size to 512 in your client, see [Tile scheme](/en/wissen/kachelschema/).

## My points end up in the wrong country. Why?

Probably longitude and latitude are swapped. mapx and GeoJSON expect `lon,lat`, see [Coordinates](/en/wissen/koordinaten/).

## Is the cadastre legally binding?

No. The [cadastre](/en/daten/kataster/) shows the DKM parcel boundaries for orientation; it does not replace a land register extract or a survey.

## Is there a map style, fonts, search or routing?

mapx provides data and elevations, no ready-made map style, no fonts, no search and no routing. Build your own style with [Style the basemap](/en/anleitungen/basiskarte-stylen/).
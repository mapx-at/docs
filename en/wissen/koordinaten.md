# Coordinates

lon,lat order, WGS84 versus Web Mercator, official sources in MGI and ETRS89.

## Order: longitude first

All mapx interfaces, TileJSON, MapLibre and GeoJSON use **`lon, lat`** (longitude, then latitude), that is east-west before north-south:

```
12.694,47.074      // lon,lat: Großglockner area
```

Many map apps and GPS devices show `lat, lon` instead. Swapped values end up in the Indian Ocean, not in Austria. The [elevation API](/en/api/hoehe/) expects `points=lon,lat;lon,lat`. Leaflet takes `[lat, lon]` in `L.latLng`, MapLibre and OpenLayers `[lon, lat]`.

## EPSG:4326 and EPSG:3857

| | EPSG:4326 (WGS84) | EPSG:3857 (Web Mercator) |
|---|---|---|
| Unit | degrees | metres |
| Used for | input and output of the APIs, GeoJSON, GPS | tile scheme of maps |
| Example | `12.694, 47.074` | `1413090, 5954161` |

Inputs to the APIs are always **WGS84 (EPSG:4326)**. Tiles are in **Web Mercator (EPSG:3857)**, see [Tile scheme](/en/wissen/kachelschema/). Clients convert between them themselves; you do not need to compute tile coordinates.

Web Mercator distances in Austria are about 50 % larger than true metres (factor `1 / cos(latitude)`). Do not measure lengths and areas in EPSG:3857; use geodesic measurement or a local system such as EPSG:31287 (MGI / Austria Lambert) or EPSG:3035 (ETRS89-LAEA).

## Official sources: MGI and ETRS89

The authorities' source data uses different systems: Austrian national survey (MGI, for example Gauss-Krüger M28/M31/M34 and Lambert) and ETRS89 (for example ETRS89-LAEA, UTM). Between MGI and WGS84/ETRS89 there is a positional offset of several metres that is corrected with an official transformation grid.

**mapx transforms correctly**: all products are converted to WGS84 and Web Mercator with the official transformation procedure. You get layers that line up without applying a transformation yourself. When you overlay your own official data in MGI, use the transformation grid of your software (for example QGIS with the BEV grid), otherwise it shifts by a few metres.

## Further reading

- [Tile scheme](/en/wissen/kachelschema/)
- [Elevation and profile](/en/api/hoehe/)
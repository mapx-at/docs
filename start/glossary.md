# Glossary of mapx and GIS terms

Source: https://docs.mapx.at/start/glossary/

> Glossary of terms used in the mapx documentation: annotations, contours, isochrones, PMTiles, Terrain-RGB and more.

**Annotation.** Extra values per route segment, for example speed or elevation. Requested with `annotations`.

**Contours.** Lines of equal elevation. Tileset `contours`.

**DEM.** Digital elevation model, here stored as Terrain-RGB tiles. See [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md).

**DSM / DTM.** Digital surface model (including buildings and trees) and digital terrain model (bare ground).

**Geocoding.** Turning an address or place name into coordinates. Reverse geocoding does the opposite.

**Glyphs.** Font data that MapLibre loads to draw labels. See [Styles API](https://docs.mapx.at/api/styles.md).

**Isochrone.** An area reachable from a point within a given time or distance.

**Orthophoto.** Rectified aerial photograph.

**PMTiles.** Single-file archive of tiles, read with HTTP range requests. The mapx styles read their tiles from PMTiles; the page must register the `pmtiles://` protocol for that (`maplibregl.addProtocol`, see [First map](https://docs.mapx.at/start/first-map.md)). Otherwise your own applications use TileJSON or `{z}/{x}/{y}`.

**Profile.** The kind of traveler for routing: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

**Sprite.** Image atlas with the icons of a style.

**Style.** A JSON document that tells the map which data to draw and how. mapx offers `streets`, `satellite`, `outdoors` and more; all styles are listed in the [style reference](https://docs.mapx.at/reference/style-spec.md).

**Tile.** A square piece of map addressed as `{z}/{x}/{y}`. See [Tile scheme](https://docs.mapx.at/reference/tile-scheme.md).

**TileJSON.** A small JSON description of a tileset: URL template, zoom range, bounds, attribution.

**Tileset.** A named collection of tiles, for example `streets` or `contours`. See [Tilesets](https://docs.mapx.at/tilesets/index.md).

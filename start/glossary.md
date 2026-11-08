# Glossary

**Annotation.** Extra values per route segment, for example speed or elevation. Requested with `annotations`.

**Cadastral municipality (KG).** Austrian land registry unit. A parcel number is only unique within one KG.

**Contours.** Lines of equal elevation. Tileset `contours`.

**DEM.** Digital elevation model, here stored as Terrain-RGB tiles. See [Terrain-RGB](/reference/terrain-rgb/).

**DSM / DTM.** Digital surface model (including buildings and trees) and digital terrain model (bare ground).

**Geocoding.** Turning an address or place name into coordinates. Reverse geocoding does the opposite.

**Glyphs.** Font data that MapLibre loads to draw labels. See [Styles API](/api/styles/).

**Isochrone.** An area reachable from a point within a given time or distance.

**Orthophoto.** Rectified aerial photograph.

**PMTiles.** Single-file archive of tiles, read with HTTP range requests. Every tileset is available as one.

**Profile.** The kind of traveler for routing: `driving`, `driving-truck`, `cycling`, `walking`.

**Sprite.** Image atlas with the icons of a style.

**Style.** A JSON document that tells the map which data to draw and how. mapx offers `streets`, `satellite` and `outdoors`.

**Tile.** A square piece of map addressed as `{z}/{x}/{y}`. See [Tile scheme](/reference/tile-scheme/).

**TileJSON.** A small JSON description of a tileset: URL template, zoom range, bounds, attribution.

**Tileset.** A named collection of tiles, for example `streets` or `cadastre`. See [Tilesets](/tilesets/).


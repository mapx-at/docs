# Glossary of mapx and GIS terms

Source: https://docs.mapx.at/reference/glossary/

> Glossary of terms used in the mapx documentation: annotations, contours, isochrones, PMTiles, Terrain-RGB and more.

**Annotation.** Extra values per route segment, for example speed or elevation. Requested with `annotations`.

**Attribution.** Required credit for the data sources (CC BY 4.0). Every JSON response carries it in the `attribution` field, shaped `mapx.at; <source> (<license>); …`. See [Attribution and license](https://docs.mapx.at/reference/attribution.md).

**Closures.** Active full closures from the traffic reports (EVIS/ASFINAG) that directions, matrix, isochrone and optimization avoid automatically. The `closures` block (`applied`, `count`, `binding`, `binding_count`, `fetched`) tells which closures applied. See [Navigation](https://docs.mapx.at/guides/navigation.md).

**Contours.** Lines of equal elevation. Tileset `contours`.

**DEM.** Digital elevation model, here stored as Terrain-RGB tiles. See [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md).

**DSM / DTM.** Digital surface model (including buildings and trees) and digital terrain model (bare ground).

**Geocoding.** Turning an address or place name into coordinates. Reverse geocoding does the opposite.

**Glyphs.** Font data that MapLibre loads to draw labels. See [Styles API](https://docs.mapx.at/api/styles.md).

**Isochrone.** An area reachable from a point within a given time or distance.

**Map matching.** Snapping a recorded GPS trace to the road graph. See [Map Matching API](https://docs.mapx.at/api/map-matching.md).

**Matrix.** Durations and distances between many origins and destinations in one request. See [Matrix API](https://docs.mapx.at/api/matrix.md).

**`modes`.** Filter and field of the public transport endpoints, for example `rail`, `bus`, `tram`, `subway`. See [Public transport API](https://docs.mapx.at/api/transit.md).

**Origin.** Where a browser request comes from (`https://example.com`). Public tokens (`mx_pk_`) only work from allowed origins. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md).

**Orthophoto.** Rectified aerial photograph.

**Place.** The shared response format of all search endpoints (geocoding, Search Box, autofill): a GeoJSON feature with `properties.type`, `name`, `place_formatted`, `relevance`, `accuracy` and `address`. See [Geocoding API](https://docs.mapx.at/api/geocoding.md).

**PMTiles.** Single-file archive of tiles, read with HTTP range requests. The mapx styles read their tiles from PMTiles; the page must register the `pmtiles://` protocol for that (`maplibregl.addProtocol`, see [First map](https://docs.mapx.at/start/first-map.md)). Otherwise your own applications use TileJSON or `{z}/{x}/{y}`.

**POI.** Point of interest, a place such as a fuel station, museum or railway station. Searchable by category with the [Search Box API](https://docs.mapx.at/api/search-box.md), on the map in the `poi_p` layer of `streets`.

**Problem Details.** Error format after RFC 9457: JSON with `type`, `title`, `status`, `detail` and the machine-readable `code` (for example `unknown-parameter`). `detail` is always English; evaluate `code`. See [Errors and limits](https://docs.mapx.at/api/errors.md).

**Profile.** The kind of traveler for routing: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`, for the isochrone also `transit`.

**Sprite.** Image atlas with the icons of a style.

**Style.** A JSON document that tells the map which data to draw and how. mapx offers `streets`, `satellite`, `outdoors` and more; all styles are listed in the [style reference](https://docs.mapx.at/reference/style-spec.md).

**Tile.** A square piece of map addressed as `{z}/{x}/{y}`. See [Tile scheme](https://docs.mapx.at/reference/tile-scheme.md).

**TileJSON.** A small JSON description of a tileset: URL template, zoom range, bounds, attribution.

**Tileset.** A named collection of tiles, for example `streets` or `contours`. See [Tilesets](https://docs.mapx.at/tilesets/index.md).

**Token.** Access key for the API: `mx_pk_…` public (websites and apps, bound to origins), `mx_sk_…` secret (servers), `mx_mcp_…` for AI assistants via the MCP server. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md).

**`transit`.** Public transport profile of the isochrone: walking plus public transport from `depart_at`. Journeys by public transport come from the [Public transport API](https://docs.mapx.at/api/transit.md).

**Zone.** Group of endpoints sharing a rate limit, for example `search`, `route`, `heavy`, `tiles`. Every response names it in the `RateLimit-Policy` header. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md).

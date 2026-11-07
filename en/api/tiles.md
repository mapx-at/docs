# XYZ tiles

All tile sets as XYZ tiles via /v1/tiles/{tileset}/{z}/{x}/{y}.{ext}, with TileJSON, caching and ETag.

Every tile set (base map, cadastre, contours, DTM, DSM, orthophoto, flight blocks) is served from the same address, in the shape of the Mapbox Tiles API. No key, CORS open.

**GET https://api.mapx.at/v1/tiles/{tileset}/{z}/{x}/{y}.{ext}**

<ParamTable params={[
  { name: "tileset", type: "string", required: true, desc: "Tile set, names see <a href=\"/en/api/kacheln/\">Tiles</a> (e.g. <code>mapx-basis</code>, <code>mapx-kataster</code>, <code>hoehenlinien</code>, <code>dtm</code>, <code>dsm</code>, <code>dop-ortho</code>)." },
  { name: "z, x, y", type: "integer", required: true, desc: "Zoom, column, row in the XYZ scheme (origin top left)." },
  { name: "ext", type: "string", required: true, desc: "Matches the tile set: <code>pbf</code> (alias <code>mvt</code>) for vector, <code>webp</code> or <code>png</code> for raster. Any other extension gives <code>404</code>." },
]} />

The address `tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}` without `/tiles/` stays valid but is deprecated (marked `deprecated` in the OpenAPI); new clients use `/v1/tiles/…`.

## Tileset list

**GET https://api.mapx.at/v1/tiles**

All available tilesets in `tilesets[]`: `name`, `aliases` (equivalent names such as `dtm`; the former German names such as `basis` stay valid as aliases until 2027-05-31), `type` (`vector`, `raster`, `dem`), `format`, `minzoom`, `maxzoom`, `bounds`, `tilejson` (URL), `attribution`, `version`, `stand` (data date) and `built` (build time). The names also work for [Tilequery](/en/api/tilequery/).

## TileJSON

**GET https://api.mapx.at/v1/tiles/{tileset}.json**

The `tiles` field of the TileJSON points at the tile address above, plus `minzoom`, `maxzoom`, `bounds`, `center` and, for vector, `vector_layers`. Details: <a href="/en/api/tilejson/">TileJSON</a>.

## Responses

| Status | Meaning |
|---|---|
| `200` | Tile. Vector tiles carry `Content-Encoding: gzip` when the client sends `Accept-Encoding: gzip`, otherwise they are decompressed. |
| `304` | `If-None-Match` matches the `ETag`. |
| `400` | Row is not a number. |
| `404` | Unknown tile set, extension does not match, zoom or tile out of range, tile empty. Errors are `application/problem+json`. |

The `ETag` derives from the file hash of the tile set, `Cache-Control: public, max-age=86400`. A missing tile is not a service error (as with Mapbox): clients skip it.

> For MapLibre and self-hosting the <a href="/en/api/kacheln/">PMTiles files</a> are more efficient: one request reads only the ranges needed.
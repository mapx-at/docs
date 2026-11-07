# Tiles

Single tiles via /v1/{tileset}/{z}/{x}/{y}.{ext} or the whole PMTiles file via HTTP range requests.

Every product is available in two ways: as single tiles (XYZ, for any map client) and as a PMTiles file (one file, the client reads only the tiles it needs via HTTP range requests). Content and quality are identical.

| | XYZ | PMTiles |
|---|---|---|
| Best for | Leaflet, OpenLayers, QGIS, apps, servers | MapLibre, self-hosting, offline |
| Address | `/v1/{tileset}/{z}/{x}/{y}.{ext}` | `<file>.pmtiles` |
| Stability | tileset name always points to the latest release | file with the release date in its name, never changes |

## Retrieve a tile

**GET https://tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}**

<ParamTable params={[
  { name: "tileset", type: "string", required: true, desc: "Product.", values: [
    { v: "dtm", d: "<a href=\"/en/daten/dtm/\">DTM</a>, Terrain-RGB, <code>.webp</code> (also <code>relief-terrainrgb</code>)" },
    { v: "dsm", d: "<a href=\"/en/daten/dsm/\">DSM</a>, Terrain-RGB, <code>.webp</code> (also <code>relief-dsm-terrainrgb</code>)" },
    { v: "dop-ortho", d: "<a href=\"/en/daten/orthophoto/\">Orthophoto</a>, Image, <code>.webp</code>" },
    { v: "mapx-basis", d: "<a href=\"/en/daten/basiskarte/\">Basemap</a>, Vector, <code>.pbf</code>" },
    { v: "hoehenlinien", d: "<a href=\"/en/daten/hoehenlinien/\">Contours</a>, Vector, <code>.pbf</code>" },
    { v: "mapx-kataster", d: "<a href=\"/en/daten/kataster/\">Cadastre</a>, Vector, <code>.pbf</code>" },
    { v: "dop-flugblock", d: "<a href=\"/en/daten/flugbloecke/\">Flight blocks</a>, Vector, <code>.pbf</code>" }
  ] },
  { name: "z", type: "integer", required: true, desc: "Zoom level within the product range (see the product page). Outside it the service answers <code>404</code>." },
  { name: "x", type: "integer", required: true, desc: "Column, 0 to 2<sup>z</sup>−1 from west to east." },
  { name: "y", type: "integer", required: true, desc: "Row, 0 to 2<sup>z</sup>−1 from north to south (XYZ scheme, not TMS)." },
  { name: "ext", type: "string", required: true, desc: "File extension, must match the product, otherwise <code>404</code>.", values: [
    { v: "webp", d: "Image and elevation tiles (DTM, DSM lossless; orthophoto)" },
    { v: "pbf", d: "Vector tiles (Mapbox Vector Tile 2.1), gzip if the client accepts it" },
  ] },
]} />

### Example

<Tabs syncKey="client">
  <TabItem label="curl">
<Code lang="bash" code={`curl -s -o tile.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp
curl -s --compressed -o tile.pbf https://tiles.mapx.at/v1/mapx-basis/14/8860/5730.pbf`} />
  </TabItem>
  <TabItem label="Leaflet">
<Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, maxNativeZoom: 18, maxZoom: 20,
  attribution: "Orthophoto: BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="OpenLayers">
<Code lang="js" code={`new ol.layer.Tile({ source: new ol.source.XYZ({
  url: "https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp", tileSize: 512, maxZoom: 18
}) });`} />
  </TabItem>
</Tabs>

### Response

| Header | Meaning |
|---|---|
| `Content-Type` | `image/webp` or `application/vnd.mapbox-vector-tile` |
| `ETag` | tile version; with <code>If-None-Match</code> the service answers <code>304</code> |
| `Cache-Control` | `public, max-age=3600` |

If there is no data at this position the service answers <code>404</code> as <code>problem+json</code> (like Mapbox, not <code>204</code>). Clients treat this as an empty tile.

## PMTiles file

**GET https://tiles.mapx.at/<file>.pmtiles**

The file supports HTTP range requests. MapLibre reads it with the [pmtiles](https://docs.protomaps.com/pmtiles/maplibre) protocol directly from the browser; the current file name is on every product page. Because a file never changes it can be cached or mirrored indefinitely.

<Code lang="js" code={`import { Protocol } from "pmtiles";
maplibregl.addProtocol("pmtiles", new Protocol().tile);
// source: { type: "raster-dem", url: "pmtiles://https://tiles.mapx.at/<file>.pmtiles", encoding: "mapbox", tileSize: 512 }`} />

Self-hosting: [Host PMTiles yourself](/en/anleitungen/pmtiles-selbst-hosten/).
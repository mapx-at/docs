# TileJSON

TileJSON 3.0 per product at /v1/{tileset}.json: tile URL, zoom range, bounds and attribution in one response.

TileJSON describes a product so that map clients can add it without further settings: MapLibre, Mapbox GL, OpenLayers and QGIS read zoom range, bounds and attribution from it. The URL stays the same when a product gets a new release.

## Retrieve TileJSON

**GET https://tiles.mapx.at/v1/{tileset}.json**

<ParamTable params={[
  { name: "tileset", type: "string", required: true, desc: "Product, values as for <a href=\"/en/api/kacheln/\">Tiles</a>." },
]} />

<Code lang="js" code={`map.addSource("ortho", { type: "raster", url: "https://tiles.mapx.at/v1/orthophoto.json", tileSize: 512 });`} />

## Response

<ParamTable kind="response" params={[
  { name: "tilejson", type: "string", desc: "Spec version: <code>3.0.0</code>" },
  { name: "tiles", type: "string[]", desc: "Tile URL template <code>{z}/{x}/{y}</code>" },
  { name: "minzoom, maxzoom", type: "integer", desc: "Zoom range with stored tiles; above it the client overzooms" },
  { name: "bounds", type: "number[4]", desc: "Bounds <code>[west, south, east, north]</code> in degrees" },
  { name: "center", type: "number[3]", desc: "Suggested initial view <code>[lon, lat, zoom]</code>" },
  { name: "scheme", type: "string", desc: "Tile scheme", values: [{ v: "xyz", d: "row 0 in the north" }] },
  { name: "tileSize", type: "integer", desc: "Tile size in pixels for raster and elevation tiles: <code>512</code>" },
  { name: "encoding", type: "string", desc: "Elevation tiles only: height encoding", values: [{ v: "mapbox", d: "<a href=\"/en/konzepte/terrain-rgb/\">Terrain-RGB</a>, 0.1 m" }] },
  { name: "vector_layers", type: "object[]", desc: "Vector tiles only: layers with <code>id</code>, <code>minzoom</code>, <code>maxzoom</code> and <code>fields</code> (attribute name → type)" },
  { name: "attribution", type: "string", desc: "Required credit under CC BY 4.0 as HTML" },
  { name: "name, description, version", type: "string", desc: "Name, description and release of the product" },
]} />

<Code lang="json" code={`{
  "tilejson": "3.0.0",
  "name": "relief-terrainrgb",
  "tiles": ["https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp"],
  "minzoom": 5, "maxzoom": 16,
  "bounds": [9.5306, 46.3697, 17.1661, 49.0235],
  "scheme": "xyz", "tileSize": 512, "encoding": "mapbox",
  "attribution": "Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0"
}`} />

The response carries <code>ETag</code> and <code>Cache-Control</code>.
# Tile scheme

Web Mercator, z/x/y, 512-pixel tiles, resolution per zoom level and overzoom.

All mapx tiles follow the common web map scheme. With MapLibre, Leaflet, OpenLayers or QGIS you do not need to configure anything; for your own requests and calculations, here is what matters.

## Projection and address

The world is projected in **Web Mercator** (EPSG:3857) onto a square and divided into zoom levels. At level `z` the square has `2^z × 2^z` tiles. A tile is addressed as `z/x/y`: `x` counts from west to east, `y` from **north to south** (XYZ scheme, not TMS), both starting at 0.

Tile for a point `lon, lat` (degrees) at level `z`:

```js
function tile(lon, lat, z) {
  const n = 2 ** z;
  const x = Math.floor(((lon + 180) / 360) * n);
  const r = (lat * Math.PI) / 180;
  const y = Math.floor(((1 - Math.log(Math.tan(r) + 1 / Math.cos(r)) / Math.PI) / 2) * n);
  return { z, x, y };
}
tile(12.694, 47.074, 12);   // { z: 12, x: 2192, y: 1439 }
```

A tile is served by `https://tiles.mapx.at/v1/tiles/{tileset}/{z}/{x}/{y}.{ext}`, see [Tiles](/api/tiles/).

## 512-pixel tiles

mapx raster tiles (elevation models, orthophoto) are **512 × 512 pixels**, not 256. A 512 tile covers the same area as four 256 tiles one level deeper. Clients need the right setting:

- MapLibre: `tileSize: 512` (automatic with TileJSON).
- Leaflet: `tileSize: 512, zoomOffset: -1`.
- OpenLayers: `tileSize: 512` on the source.
- QGIS: tile resolution 512.

If you forget it, the map looks off by one zoom level and blurry or too fine.

## Resolution per zoom level

Resolution depends on latitude. The table applies to 47.5° N, the middle of Austria:

<Aufloesung lang="en" />

One level deeper halves the pixel size. The elevation models exist up to z16, the orthophoto up to z18.

## Overzoom

If a level is higher than the stored one, the **client scales up** the tiles of the highest level (overzoom). The image does not get sharper, but the map stays usable. In MapLibre set `maxzoom` of the source to the highest existing level (automatic from TileJSON); the map itself may zoom further. Requests outside the zoom range return `404`, see [Errors and limits](/api/errors/).

> Vector tiles are not redrawn on overzoom; the geometry of the highest level is stretched. For the cadastre (z16 only) set `minzoom: 16` in the style.

## Further reading

- Elevation values in tiles: [Terrain-RGB](/reference/terrain-rgb/).
- Coordinates and projections: [Coordinates](/reference/coordinates/).
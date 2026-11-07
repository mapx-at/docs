# Static Tiles

Rendered raster tiles for map, aerial imagery and terrain, for Leaflet, OpenLayers and QGIS.

The endpoint serves pre-rendered raster tiles of a style, in the shape of the Mapbox Static Tiles API. It lets clients without vector support show the map, aerial imagery and terrain (Leaflet, OpenLayers, QGIS, maps in reports). No key, CORS open.

**GET https://api.mapx.at/v1/styles/{id}/tiles/{tilesize}/{z}/{x}/{y}[@2x][.png|.webp]**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Style: <code>streets</code>, <code>streets-dawn</code>, <code>streets-dusk</code>, <code>streets-night</code>, <code>satellite</code>, <code>satellite-streets</code> or <code>outdoors</code>." },
  { name: "tilesize", type: "integer", required: true, desc: "<code>256</code> (standard XYZ, map zoom <code>z-1</code>) or <code>512</code> (map zoom <code>z</code>, labels look smaller)." },
  { name: "z, x, y", type: "integer", required: true, desc: "Tile in the XYZ scheme (origin top left)." },
  { name: "@2x", type: "suffix", desc: "Double pixel count (512 or 1024 px) for the same area." },
  { name: "png / webp", type: "suffix", desc: "Image format, default PNG." },
]} />

## Tile edges

The area is rendered with a 96 pixel margin and cropped to the tile. Lines, areas and shading therefore continue across tile edges without a seam, and labels are not cut at the tile edge. Label placement is computed per tile; a name near an edge can therefore sit at a slightly different place in the neighbouring tile. For labels without duplicates use the style as a vector map (<a href="/en/api/styles/">Styles</a>).

## Responses and limits

`404` for tiles without data (outside Austria) and below the minimum zoom (`satellite`, `satellite-streets` and `outdoors` from map zoom 5), invalid size, extension or tile number and unknown styles. `ETag` and `Cache-Control: public, max-age=86400`, `If-None-Match` gives `304`. Rendering is expensive and counts several times against the rate limit (<a href="/en/api/fehler/">Errors and limits</a>); repeated requests are served from the cache.

## Leaflet

```js
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/256/{z}/{x}/{y}@2x', {
  tileSize: 256,
  detectRetina: false, // @2x ist schon in der URL / @2x is already in the URL
  maxZoom: 22,
  attribution: '© mapx.at, BEV, GIP.at'
}).addTo(map);
```

For 512 px tiles set `tileSize: 512` and `zoomOffset: -1` and use `/tiles/512/`.

## QGIS

Browser, right-click **XYZ Tiles**, **New Connection**:

| | |
|---|---|
| Name | mapx Karte |
| URL | `https://api.mapx.at/v1/styles/streets/tiles/256/{z}/{x}/{y}` |
| Max. zoom | 22 |

For `satellite` and `outdoors` enter min. zoom 5. The layer works like any raster layer in layouts and reports.

## Single tile

```sh
curl -o kachel.png 'https://api.mapx.at/v1/styles/satellite/tiles/256/15/17767/11367'
```
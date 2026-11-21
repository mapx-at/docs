# mapx maps with OpenLayers

Source: https://docs.mapx.at/guides/openlayers/

> Add the mapx basemap to OpenLayers: rendered raster tiles of the style, token, attribution, your own GeoJSON and the limits compared with MapLibre.

**Goal:** a mapx map in an existing OpenLayers application (10.11.0), no build, with a free public token (see [Access and limits](https://docs.mapx.at/start/access-and-limits.md)).

## Add the basemap

The mapx styles are MapLibre styles with vector tiles. OpenLayers gets the same style rendered as raster tiles ([Static Tiles](https://docs.mapx.at/api/styles.md)): `ol/source/XYZ` with tile size 512. Styles: `streets` (and variants), `light`, `dark`, `satellite`, `satellite-streets`, `outdoors`, `outdoors-winter`; not available as raster tiles are `traffic-day`, `traffic-night`, `satellite-current`, the relief styles and the `-3d` styles.

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
<div id="map" style="height:70vh"></div>
<script>
const TOKEN = 'YOUR_TOKEN';
const map = new ol.Map({
  target: 'map',
  layers: [new ol.layer.Tile({ source: new ol.source.XYZ({
    url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN,
    tileSize: 512, maxZoom: 20, attributionsCollapsible: false,
    attributions: '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
  }) })],
  view: new ol.View({ center: ol.proj.fromLonLat([16.37, 48.21]), zoom: 13 }),
});
</script>
```

The token goes into the tile URL as `access_token`; API calls with `fetch` send it as the header `Authorization: Bearer …`.

## Attribution

The data is licensed CC BY 4.0 and users must be able to reach the source on the map. Raster tiles have no TileJSON for OpenLayers to read the text from: set `attributions` yourself. `attributionsCollapsible: false` shows the text permanently; without it OpenLayers puts it behind an "i" button, like the MapLibre examples in these docs. The text per source is listed in [Attribution and license](https://docs.mapx.at/reference/attribution.md).

## Your own GeoJSON

API responses (route, isochrone …) are GeoJSON in WGS84. `ol.format.GeoJSON` reads them into the map projection:

```js
const source = new ol.source.Vector();
map.addLayer(new ol.layer.Vector({ source }));
source.addFeatures(new ol.format.GeoJSON().readFeatures(geojson, { featureProjection: 'EPSG:3857' }));
```

## Limits

- No vector style: the basemap arrives as an image, labels and layers cannot be changed one by one. Use [MapLibre](https://docs.mapx.at/guides/maps.md) for that.
- No 3D terrain and no 3D buildings, these are only available in MapLibre.
- Raster tiles count in the rate limit zone `static` (5 requests per second per IP address, briefly up to 60 at once, plus the per-minute limit of 6000 per token, see [Access and limits](https://docs.mapx.at/start/access-and-limits.md)). Ordinary panning and zooming stays well below that; only requesting new tiles faster for a sustained period returns `429`.

## Examples

Examples with an OpenLayers version are listed in the [overview](https://docs.mapx.at/examples/index.md) (filter "OpenLayers"), for instance [Embed a map](https://docs.mapx.at/examples/embed-map.md#ol) and [Route](https://docs.mapx.at/examples/route.md#ol).

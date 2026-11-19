# mapx maps with Leaflet

Source: https://docs.mapx.at/guides/leaflet/

> Add the mapx basemap to Leaflet: rendered raster tiles of the style, token, attribution, your own GeoJSON and the limits compared with MapLibre.

**Goal:** a mapx map in an existing Leaflet application (1.9.4), no build and no plugin, with a free public token (see [Access and limits](https://docs.mapx.at/start/access-and-limits.md)).

## Add the basemap

Leaflet does not draw vector tiles. The mapx styles are therefore also available rendered as raster tiles ([Static Tiles](https://docs.mapx.at/api/styles.md)), added with `L.tileLayer`. The tiles are 512 pixels: `tileSize: 512` and `zoomOffset: -1` keep the scale of ordinary 256 pixel tiles. Styles: `streets` (and variants), `satellite`, `satellite-streets`, `outdoors`.

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<div id="map" style="height:70vh"></div>
<script>
const TOKEN = 'YOUR_TOKEN';
const map = L.map('map').setView([48.21, 16.37], 13);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, {
  tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21,
  attribution: '<a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
}).addTo(map);
</script>
```

The token goes into the tile URL as `access_token`; API calls with `fetch` send it as the header `Authorization: Bearer …`.

## Attribution

The data is licensed CC BY 4.0 and users must be able to reach the source on the map. Leaflet shows the `attribution` text in the bottom right corner; the text per source is listed in [Attribution and license](https://docs.mapx.at/reference/attribution.md).

## Your own GeoJSON

API responses (route, isochrone, parcel …) are GeoJSON and go straight into `L.geoJSON`:

```js
const layer = L.geoJSON(null, { style: { color: '#2346d8', weight: 4 } }).addTo(map);
layer.addData(geojson);
```

## Limits

- No vector style: the basemap arrives as an image, labels and layers cannot be changed one by one. Use [MapLibre](https://docs.mapx.at/guides/maps.md) for that.
- No 3D terrain and no 3D buildings, these are only available in MapLibre.
- Raster tiles count in the rate limit zone `static` (5 requests per second per IP, briefly up to 60 at once, see [Access and limits](https://docs.mapx.at/start/access-and-limits.md)). Ordinary panning and zooming stays well below that; only requesting new tiles faster for a sustained period returns `429`.

## Examples

Examples with a Leaflet version are listed in the [overview](https://docs.mapx.at/examples/index.md) (filter "Leaflet"), for instance [Embed a map](https://docs.mapx.at/examples/embed-map.md#leaflet), [Route](https://docs.mapx.at/examples/route.md#leaflet) and [Parcel on click](https://docs.mapx.at/examples/parcel-on-click.md#leaflet).

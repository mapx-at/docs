# Quick start

Your first map with MapLibre and PMTiles in five minutes, no key.

export const t = data.tilesets.relief;
export const url = `https://tiles.mapx.at/${t.datei}`;

All mapx tiles are available as [PMTiles](https://docs.protomaps.com/pmtiles/) files on `tiles.mapx.at`. The browser reads only the tiles it needs
via HTTP range. There is no key and no sign-up, CORS is open.

## 1. Page

<Code lang="html" title="index.html" code={`<!doctype html>
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.16.0/dist/maplibre-gl.css">
<script src="https://unpkg.com/maplibre-gl@6.16.0/dist/maplibre-gl.js"></script>
<script src="https://unpkg.com/pmtiles@4.5.0/dist/pmtiles.js"></script>
<div id="map" style="height:100vh"></div>
<script>
  maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
  new maplibregl.Map({
    container: "map",
    center: [12.694, 47.074], zoom: 11, pitch: 60,
    style: {
      version: 8,
      sources: { dem: { type: "raster-dem", url: "pmtiles://${url}", tileSize: 512, encoding: "mapbox", maxzoom: 16 } },
      terrain: { source: "dem", exaggeration: 1.2 },
      layers: [{ id: "hill", type: "hillshade", source: "dem" }]
    }
  });
</script>`} />

Save the file as `index.html` and open it with any static web server: done, shaded 3D terrain around the Großglockner.

## 2. Next

- Live map with elevation per click, code examples for Leaflet, OpenLayers, QGIS and more: [DTM terrain model](/en/daten/dtm/).
- Elevation via HTTP instead of tile decoding: [API](/en/api/hoehe/).
- How elevation becomes color: [Terrain-RGB](/en/konzepte/terrain-rgb/).
- Names with reference date change with every new release. The current name is on the product page and in the [TileJSON](/en/api/tilejson/).
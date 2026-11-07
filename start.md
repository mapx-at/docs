# Schnellstart

In fünf Minuten zur ersten Karte mit MapLibre und PMTiles, ohne Schlüssel.

export const t = data.tilesets.relief;
export const url = `https://tiles.mapx.at/${t.datei}`;

Alle mapx-Kacheln liegen als [PMTiles](https://docs.protomaps.com/pmtiles/)-Dateien auf `tiles.mapx.at`. Der Browser liest per HTTP-Range
nur die Kacheln, die er braucht. Es gibt keinen Schlüssel und keine Anmeldung, CORS ist offen.

## 1. Seite

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

Die Datei in `index.html` speichern, mit einem beliebigen statischen Server öffnen (zum Beispiel `npx serve`), fertig: schattiertes 3D-Gelände rund um den Großglockner.

## 2. Weiter

- Live-Karte mit Höhe pro Klick, Codebeispiele für Leaflet, OpenLayers, QGIS und Python: [DTM Geländemodell](/daten/dtm/).
- Höhe per HTTP statt Kachel-Dekodierung: [API](/api/hoehe/).
- Wie die Höhe in die Farbe kommt: [Terrain-RGB](/konzepte/terrain-rgb/).
- Dateinamen enthalten den Stichtag und ändern sich bei jedem neuen Stand. Der aktuelle Name steht auf der Produktseite und im [TileJSON](/api/tilejson/).
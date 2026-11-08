# 3D-Gelände mit Orthophoto

Eine 3D-Karte mit echtem Höhenrelief, Schummerung und Luftbild darüber, mit MapLibre GL JS und kostenlosem Token.

**Ziel:** Eine Karte, auf der das Orthophoto über dem echten Gelände liegt, plastisch schattiert und im Browser kippbar.

**Voraussetzungen:** ein Browser, ein beliebiger statischer Webserver (zum Beispiel `npx serve`). Ein kostenloser API-Token ([Zugang und Limits](/de/start/access-and-limits/)).

Verwendet werden das [DTM](/de/tilesets/terrain-rgb/) (Höhe, Typ `raster-dem`) und das [Orthophoto](/de/tilesets/orthophoto/) (Bild, Typ `raster`). Beide Tilesets binden Sie über [TileJSON](/de/api/tiles/) ein, ohne einen Dateinamen zu kennen.

<Steps>

1. **Seite anlegen.** Datei `index.html` mit MapLibre GL JS und einem Kartencontainer:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
   <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
   <style>html, body, #map { margin: 0; height: 100%; }</style>
   <div id="map"></div>
   ```

2. **Quellen deklarieren.** Das Höhenmodell als `raster-dem`, das Orthophoto als `raster`:

   ```js
   const quellen = {
     dem:   { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", encoding: "mapbox" },
     ortho: { type: "raster",     url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 }
   };
   ```

3. **Ebenen schichten.** Unten das Luftbild, darüber die Schummerung (`hillshade`) mit schwacher Wirkung, damit das Bild erkennbar bleibt:

   ```js
   const ebenen = [
     { id: "ortho", type: "raster", source: "ortho" },
     { id: "schummerung", type: "hillshade", source: "dem",
       paint: { "hillshade-exaggeration": 0.4, "hillshade-shadow-color": "#000000" } }
   ];
   ```

4. **Gelände einschalten.** `terrain` hebt alle Ebenen auf die Höhe des DTM. `pitch` kippt die Ansicht:

   ```html
   <script>
     const map = new maplibregl.Map({
       container: "map",
       center: [12.694, 47.074], zoom: 12, pitch: 65, bearing: 20,
       maxPitch: 80,
       style: {
         version: 8,
         sources: quellen,
         layers: ebenen,
         terrain: { source: "dem", exaggeration: 1.3 }
       }
     });
     map.addControl(new maplibregl.NavigationControl({ visualizePitch: true }));
   </script>
   ```

   Die Konstanten `quellen` und `ebenen` aus den Schritten 2 und 3 müssen im selben `<script>` vor `new maplibregl.Map` stehen.

5. **Starten.** `npx serve` im Ordner, die ausgegebene Adresse öffnen. Mit gedrückter rechter Maustaste ziehen kippt und dreht die Karte.

</Steps>

## Ergebnis

Das Luftbild liegt auf dem Gelände rund um den Großglockner, Hänge sind zusätzlich schattiert. Das Orthophoto reicht bis z18 (20 cm), das DTM bis z16 (Rasterweite 1 m); MapLibre vergrößert darüber hinaus automatisch.

> Mit `exaggeration` überhöhen Sie das Relief. `1.0` entspricht der Natur; bei flachem Gelände wirken `1.5` bis `2` plastischer.

## Nächste Schritte

- Höhe pro Klick auslesen: `map.queryTerrainElevation(e.lngLat)`, Details in [DTM Geländemodell](/de/tilesets/terrain-rgb/).
- Höhenlinien und Beschriftung darüber: [Höhenlinien überlagern](/de/guides/overlay-contours/).
- Höhenprofil entlang einer Linie: [Höhenprofil](/de/guides/elevation-profile/).
- Quellenangabe richtig anzeigen: [Attribution](/de/reference/attribution/).
# Eigene Karte aus der Basiskarte


**Ziel:** Eine eigene, farblich abgestimmte Karte aus den Vektorkacheln der [Basiskarte](/de/tilesets/streets/) bauen. mapx liefert die Daten, das Aussehen legt der eigene Style fest.

**Voraussetzungen:** ein Browser und ein statischer Webserver. Grundkenntnisse in MapLibre-Styles (Quellen und Ebenen).

Die Basiskarte hat 18 Layer. Dieses Beispiel nutzt fünf: `landuse_a` (Nutzung), `water_a` (Seen), `road_l` (Straßen), `building_a` (Gebäude) und `boundary_l` (Grenzen). Alle Layer und Felder stehen in der [Layertabelle](/de/tilesets/streets/).


1. **Seite und Quelle.** Die Quelle ist der TileJSON-Link, Zoombereich und Attribution kommen automatisch mit:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
   <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
   <style>html, body, #map { margin: 0; height: 100%; }</style>
   <div id="map"></div>
   <script>
     const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
     const quelle = { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" };
   ```

2. **Hintergrund und Nutzung.** Jede Fläche in `landuse_a` hat ein Feld `kind` (zum Beispiel `forest`, `farmland`, `vineyard`, `alpine_meadow`, `glacier`, `rock`, `residential`). Eine Farbe je Gruppe über einen `match`-Ausdruck:

   ```js
   const nutzung = {
     id: "nutzung", type: "fill", source: "streets", "source-layer": "landuse_a",
     paint: { "fill-color": ["match", ["get", "kind"],
       "forest", "#bcd9b0",
       ["farmland", "vineyard"], "#efe9c8",
       "alpine_meadow", "#d9e8c4",
       "glacier", "#f4f8fb",
       "rock", "#d8d4cf",
       ["residential", "garden"], "#ebe5de",
       "#e6e9dc"] }
   };
   ```

3. **Gewässer.** Seen und Teiche aus `water_a` als Fläche:

   ```js
   const wasser = {
     id: "wasser", type: "fill", source: "streets", "source-layer": "water_a",
     paint: { "fill-color": "#a9cde8" }
   };
   ```

4. **Straßen.** `road_l` hat `kind`: `highway`, `major_road`, `minor_road`, `service`, `track`, `path`. Breite mit dem Zoom wachsen lassen, Wege dünn und gestrichelt:

   ```js
   const strassenGrob = {
     id: "strassen", type: "line", source: "streets", "source-layer": "road_l",
     filter: ["in", ["get", "kind"], ["literal", ["highway", "major_road", "minor_road", "service"]]],
     layout: { "line-cap": "round", "line-join": "round" },
     paint: {
       "line-color": ["match", ["get", "kind"], "highway", "#e8a35c", "major_road", "#f0c27a", "#ffffff"],
       "line-width": ["interpolate", ["exponential", 1.5], ["zoom"], 8, 0.5, 14, 3, 16, 8]
     }
   };
   const wege = {
     id: "wege", type: "line", source: "streets", "source-layer": "road_l", minzoom: 13,
     filter: ["in", ["get", "kind"], ["literal", ["track", "path"]]],
     paint: { "line-color": "#a07a5a", "line-width": 1, "line-dasharray": [2, 1.5] }
   };
   ```

5. **Gebäude.** `building_a` beginnt bei z12 (dort zusammengefasst). Ab z15 hat jedes Gebäude Umriss:

   ```js
   const gebaeude = {
     id: "gebaeude", type: "fill", source: "streets", "source-layer": "building_a", minzoom: 13,
     paint: { "fill-color": "#cfc7bd", "fill-outline-color": "#a89f94" }
   };
   ```

6. **Grenzen.** `boundary_l` trägt `kind` (`country`, `state`, `district`, `municipality`). Staats- und Landesgrenze kräftig, Gemeinden dezent:

   ```js
   const grenzen = [
     { id: "grenze-staat", type: "line", source: "streets", "source-layer": "boundary_l",
       filter: ["==", ["get", "kind"], "country"],
       paint: { "line-color": "#7a4f8f", "line-width": 1.5 } },
     { id: "grenze-land", type: "line", source: "streets", "source-layer": "boundary_l",
       filter: ["==", ["get", "kind"], "state"],
       paint: { "line-color": "#9d86a8", "line-width": 1, "line-dasharray": [4, 2] } },
     { id: "grenze-gemeinde", type: "line", source: "streets", "source-layer": "boundary_l", minzoom: 10,
       filter: ["==", ["get", "kind"], "municipality"],
       paint: { "line-color": "#b9b0be", "line-width": 0.6, "line-dasharray": [2, 2] } }
   ];
   ```

7. **Karte zusammensetzen.** Reihenfolge von unten nach oben: Nutzung, Wasser, Straßen, Gebäude, Grenzen.

   ```js
     new maplibregl.Map({
       container: "map",
       center: [15.334, 48.227], zoom: 14,
       // Token an alle Anfragen an mapx.at hängen
       transformRequest: (url) => ({
         url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
       }),
       style: {
         version: 8,
         sources: { streets: quelle },
         layers: [
           { id: "grund", type: "background", paint: { "background-color": "#f2efe9" } },
           nutzung, wasser, strassenGrob, wege, gebaeude, ...grenzen
         ]
       }
     });
   </script>
   ```


## Ergebnis

Eine Karte im eigenen Look: Flächen nach Nutzung, Seen, Straßennetz mit Wegen, Gebäude und die Verwaltungsgrenzen. Alle Daten kommen aus einem einzigen Tileset.

## Beschriftung braucht eigene Glyphen

mapx liefert keine Schriften. Für Beschriftungen (`place_p`, `water_label_p`, `housenumber_p`, Straßennamen aus `road_l` über das Feld `name`) braucht der Style ein `glyphs`-Feld, das auf eine selbst betriebene oder gewählte Schriftquelle zeigt:

```js
style: {
  version: 8,
  glyphs: "https://demotiles.maplibre.org/font/{fontstack}/{range}.pbf",   // eigene Glyphenquelle
  sources: { streets: quelle },
  layers: [
    // ...
    { id: "orte", type: "symbol", source: "streets", "source-layer": "place_p",
      filter: ["in", ["get", "kind"], ["literal", ["city", "town", "village"]]],
      layout: { "text-field": ["get", "name"], "text-font": ["Open Sans Semibold"], "text-size": 13 },
      paint: { "text-color": "#333", "text-halo-color": "#fff", "text-halo-width": 1.5 } }
  ]
}
```

Das Beispiel nutzt die freien Demo-Glyphen von MapLibre (Schrift `Open Sans Semibold`). Für den Produktivbetrieb eine eigene Glyphenquelle verwenden; `text-font` muss einer Schrift entsprechen, die die Glyphenquelle ausliefert.

## Nächste Schritte

- Alle Layer, Felder und Werte: [Basiskarte](/de/tilesets/streets/).
- Höhenlinien und Beschriftung darüberlegen: [Höhenlinien überlagern](/de/guides/overlay-contours/).
- Karte für den Druck oder als Offline-Datei: [PMTiles selbst hosten](/de/guides/self-host-pmtiles/).


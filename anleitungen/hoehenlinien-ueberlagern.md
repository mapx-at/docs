# Höhenlinien überlagern

Höhenlinien mit Indexlinien und Höhenbeschriftung über Orthophoto oder Basiskarte legen.

**Ziel:** Höhenlinien in passender Dichte über ein Luftbild legen, Indexlinien kräftiger zeichnen und mit der Höhe beschriften.

**Voraussetzungen:** ein Browser und ein statischer Webserver. Beschriftung braucht eine Schriftquelle (Glyphen), die Sie selbst angeben, siehe [Basiskarte stylen](/anleitungen/basiskarte-stylen/#beschriftung-braucht-eigene-glyphen).

Das Tileset [Höhenlinien](/daten/hoehenlinien/) hat einen Layer `contour_l` mit den Feldern `ele` (Höhe in m), `index` (1 = Indexlinie) und `lvl` (Generalisierungsstufe 10, 12, 14, 16). Es gibt Linien ab z10.

<Steps>

1. **Seite mit Hintergrund.** Das Orthophoto als Basis, die Höhenlinien als Vektorquelle:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.16.0/dist/maplibre-gl.css">
   <script src="https://unpkg.com/maplibre-gl@6.16.0/dist/maplibre-gl.js"></script>
   <style>html, body, #map { margin: 0; height: 100%; }</style>
   <div id="map"></div>
   <script>
     const quellen = {
       ortho: { type: "raster", url: "https://tiles.mapx.at/v1/orthophoto.json", tileSize: 512 },
       linien: { type: "vector", url: "https://tiles.mapx.at/v1/contours.json" }
     };
   ```

2. **Stufe wählen.** Die Kacheln enthalten mehrere Stufen mit eigenem Linienabstand. Damit je Zoom nur die passende Stufe sichtbar ist, nach `lvl` filtern:

   | Zoom | `lvl` | Abstand | Indexlinie |
   |---|---|---|---|
   | 10–11 | 10 | 100 m | 500 m |
   | 12–13 | 12 | 50 m | 200 m |
   | 14–15 | 14 | 20 m | 100 m |
   | ab 16 | 16 | 10 m | 50 m |

   Eine Ebene je Stufe, mit `minzoom` und `maxzoom` begrenzt:

   ```js
   const stufen = [
     { lvl: 10, min: 10, max: 12 },
     { lvl: 12, min: 12, max: 14 },
     { lvl: 14, min: 14, max: 16 },
     { lvl: 16, min: 16, max: 24 }
   ];
   const linienEbenen = stufen.flatMap(({ lvl, min, max }) => [
     { id: `linie-${lvl}`, type: "line", source: "linien", "source-layer": "contour_l",
       minzoom: min, maxzoom: max,
       filter: ["all", ["==", ["get", "lvl"], lvl], ["!=", ["get", "index"], 1]],
       paint: { "line-color": "#ffd9a0", "line-width": 0.6, "line-opacity": 0.8 } },
     { id: `index-${lvl}`, type: "line", source: "linien", "source-layer": "contour_l",
       minzoom: min, maxzoom: max,
       filter: ["all", ["==", ["get", "lvl"], lvl], ["==", ["get", "index"], 1]],
       paint: { "line-color": "#ffb347", "line-width": 1.4 } }
   ]);
   ```

3. **Höhenbeschriftung.** Nur Indexlinien tragen eine Zahl. `symbol-placement: "line"` setzt den Text entlang der Linie:

   ```js
   const beschriftung = {
     id: "hoehe-text", type: "symbol", source: "linien", "source-layer": "contour_l", minzoom: 12,
     filter: ["==", ["get", "index"], 1],
     layout: {
       "symbol-placement": "line",
       "text-field": ["concat", ["to-string", ["get", "ele"]], " m"],
       "text-font": ["Open Sans Semibold"],
       "text-size": 11
     },
     paint: { "text-color": "#fff", "text-halo-color": "#7a4a00", "text-halo-width": 1.5 }
   };
   ```

   `text-font` muss eine Schrift Ihrer Glyphenquelle sein. Ohne Beschriftung entfällt diese Ebene und `glyphs`.

4. **Karte starten.**

   ```js
     new maplibregl.Map({
       container: "map",
       center: [12.694, 47.074], zoom: 14,
       style: {
         version: 8,
         glyphs: "https://demotiles.maplibre.org/font/{fontstack}/{range}.pbf",   // Ihre Glyphenquelle
         sources: quellen,
         layers: [{ id: "ortho", type: "raster", source: "ortho" }, ...linienEbenen, beschriftung]
       }
     });
   </script>
   ```

</Steps>

## Ergebnis

Über dem Luftbild liegen feine Höhenlinien, deren Abstand sich mit dem Zoom anpasst (100 m bis 10 m). Indexlinien sind kräftig und tragen die Höhe in Metern.

Für eine Karte statt Luftbild ersetzen Sie die Orthophoto-Ebene durch die Ebenen aus [Basiskarte stylen](/anleitungen/basiskarte-stylen/); die Höhenlinien kommen danach darüber.

> Ohne Filter nach `lvl` liegen beim Überzoomen mehrere Stufen übereinander. Die Linien enden unterhalb von z10; für kleinere Maßstäbe bleibt das Relief über [Schummerung](/anleitungen/gelaende-3d/).

## Nächste Schritte

- Felder und Stufen im Detail: [Höhenlinien](/daten/hoehenlinien/).
- Gelände dreidimensional: [3D-Gelände](/anleitungen/gelaende-3d/).
- Quellenangabe: [Attribution](/anleitungen/attribution/).
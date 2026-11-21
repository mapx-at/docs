# Höhenlinien über Orthofoto legen

Source: https://docs.mapx.at/de/guides/overlay-contours/

> Höhenlinien mit Indexlinien und Höhenbeschriftung über Orthofoto oder Basiskarte in MapLibre GL JS legen, mit den mapx-Höhenlinien als Vektorkacheln.

**Ziel:** Höhenlinien in passender Dichte über ein Luftbild legen, Indexlinien kräftiger zeichnen und mit der Höhe beschriften.

**Voraussetzungen:** ein Browser, ein statischer Webserver und ein kostenloser öffentlicher Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Die Beschriftung nutzt die Schriften von mapx, siehe [Basiskarte gestalten](https://docs.mapx.at/de/guides/style-the-basemap.md#beschriftung-und-schriften).

Der Kachelsatz [Höhenlinien](https://docs.mapx.at/de/tilesets/contours.md) hat eine Ebene `contour_l` mit den Feldern `ele` (Höhe in m), `index` (1 = Indexlinie) und `lvl` (Generalisierungsstufe 10, 12, 14, 16). Es gibt Linien ab z10.

1. **Seite mit Hintergrund.** Das Orthofoto als Basis, die Höhenlinien als Vektorquelle:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.css">
   <style>html, body, #map { margin: 0; height: 100%; }</style>
   <div id="map"></div>
   <script type="module">
     import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";
     const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
     // Token an alle Anfragen an mapx.at hängen (TileJSON, Kacheln, Schriften)
     const withToken = (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url });
     const quellen = {
       ortho: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 },
       linien: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" }
     };
   ```

2. **Linien zeichnen.** Je nach Zoom liefert der Kachelsatz eine Stufe (`lvl`) mit eigenem Linienabstand:

   | Zoom | `lvl` | Abstand | Indexlinie |
   |---|---|---|---|
   | 10–11 | 10 | 100 m | 500 m |
   | 12–13 | 12 | 50 m | 200 m |
   | 14–15 | 14 | 20 m | 100 m |
   | ab 16 | 16 | 10 m | 50 m |

   Jede Kachel enthält nur die Stufe, die zu ihrem Zoom passt; der Abstand ändert sich daher von selbst, es braucht keinen Filter nach `lvl`. Zwei Ebenen genügen, Linien und Indexlinien:

   ```js
   const linienEbenen = [
     { id: "linie", type: "line", source: "linien", "source-layer": "contour_l",
       filter: ["!=", ["get", "index"], 1],
       paint: { "line-color": "#ffd9a0", "line-width": 0.6, "line-opacity": 0.8 } },
     { id: "index", type: "line", source: "linien", "source-layer": "contour_l",
       filter: ["==", ["get", "index"], 1],
       paint: { "line-color": "#ffb347", "line-width": 1.4 } }
   ];
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

   `text-font` muss eine Schrift sein, die `GET /v1/fonts` auflistet, zum Beispiel `Open Sans Semibold` oder `Noto Sans Regular`. Ohne Beschriftung entfällt diese Ebene und `glyphs`.

4. **Karte starten.**

   ```js
     const map = new maplibregl.Map({
       attributionControl: false,
       container: "map",
       transformRequest: withToken,
       center: [12.694, 47.074], zoom: 14,
       style: {
         version: 8,
         glyphs: "https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf",
         sources: quellen,
         layers: [{ id: "ortho", type: "raster", source: "ortho" }, ...linienEbenen, beschriftung]
       }
     });
     map.addControl(new maplibregl.AttributionControl({ compact: true }));
     map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
   </script>
   ```

## Ergebnis

Über dem Luftbild liegen feine Höhenlinien, deren Abstand sich mit dem Zoom anpasst (100 m bis 10 m). Indexlinien sind kräftig und tragen die Höhe in Metern.

Für eine Karte statt Luftbild die Orthofoto-Ebene durch die Ebenen aus [Basiskarte gestalten](https://docs.mapx.at/de/guides/style-the-basemap.md) ersetzen; die Höhenlinien kommen danach darüber.

> Die Linien enden unterhalb von z10; für kleinere Maßstäbe bleibt das Relief über [Schummerung](https://docs.mapx.at/de/guides/3d-terrain.md).

## Nächste Schritte

- Felder und Stufen im Detail: [Höhenlinien](https://docs.mapx.at/de/tilesets/contours.md).
- Gelände dreidimensional: [3D-Gelände](https://docs.mapx.at/de/guides/3d-terrain.md).
- Quellenangabe: [Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md).

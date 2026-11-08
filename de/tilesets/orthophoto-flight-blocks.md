# Flugblöcke


Zu jedem Pixel des [Orthophotos](/de/tilesets/orthophoto/) die Fläche des Bildflugs (Operats), aus dem es stammt, mit Flugzeitraum.



## Kennwerte



## Live-Karte



## Einbinden

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json" } },
        layers: [{ id: "dop_flugblock", type: "line", source: "v", "source-layer": "dop_flugblock", paint: { "line-color": "#b5651d" } }]
      }
    });
    ```
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://https://tiles.mapx.at/dop-flugblock-20251003-0d0d122e.pmtiles"` (Protokoll wie im [Schnellstart](/de/start/)).
    **QGIS**

    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
    curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf
    ```
  
## Layer-Referenz

### dop_flugblock



## Quelle und Aktualität

- Quelle: BEV „Aktualität Orthophoto Operate Farbe und Infrarot“.
- Die Flächen zeigen, wo ein Operat im Orthophoto **tatsächlich** zu sehen ist, nicht die ursprünglichen Umrisse. Ein Operat kann mehrere Flächen haben.

## Hinweise

- Ein Flugblock ist ein Operat, kein einzelner Flugtag: Es gibt kein Tagesdatum, nur den Zeitraum des Operats; ein Operat kann bis zu drei Monate dauern.


## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

{{< attribution ts="dop_flugblock" >}}

Mehr: [Lizenz](/de/reference/attribution/).


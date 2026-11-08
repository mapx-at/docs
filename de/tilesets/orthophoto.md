# Orthophoto


Das Orthophoto des BEV, neu gekachelt in Web-Mercator bis z18 (20 cm, die native Auflösung der Quelle). Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](/de/tilesets/orthophoto-flight-blocks/).



## Kennwerte



## Live-Karte



Klick zeigt den Flugblock am Punkt (Operat, Flugzeitraum).

## Einbinden

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { ortho: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 } },
        layers: [{ id: "ortho", type: "raster", source: "ortho" }]
      }
    });
    ```
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://https://tiles.mapx.at/dop-ortho-20251003-272a54b3.pmtiles"` (Protokoll wie im [Schnellstart](/de/start/)).
    **Leaflet**

    ```js
    L.tileLayer("https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp", {
      tileSize: 512, zoomOffset: -1, crossOrigin: true,
      attribution: "BEV, CC BY 4.0"
    }).addTo(map);
    ```
    **QGIS**

    XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp`. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto.json
    curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/orthophoto/14/8860/5730.webp
    ```
  
## Quelle und Aktualität

- Quelle: BEV **Digitales Orthophoto (DOP) RGB**, 20 cm, Aufnahmen 2022 bis 2025.
- **Ein Flug je Pixel:** Wo sich Befliegungen überlappen, gewinnt die mit dem späteren Ende des Flugzeitraums. Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](/de/tilesets/orthophoto-flight-blocks/).

## Hinweise

- Ganz Österreich ist bedeckt.
- Zwischen Bildflügen sind Farbnähte sichtbar, es gibt keine Farbangleichung.
- An den Grenzen zu DE, CH, LI, IT, SI, CZ und SK reicht das Bild teils mehr als 3 km ins Ausland; an der Grenze zu Ungarn endet es an der Grenze.
- Unter z18 mischen sich an Blockgrenzen Pixel beider Flüge; eindeutig ist die Zuordnung nur in z18.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

{{< attribution ts="dop" >}}

Mehr: [Lizenz](/de/reference/attribution/).


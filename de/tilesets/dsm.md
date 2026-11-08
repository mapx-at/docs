# DSM Oberflächenmodell


Das DSM ist das Gegenstück zum [DTM](/de/tilesets/terrain-rgb/): Es zeigt die Oberfläche mit Gebäuden, Bäumen und Brücken. Kodierung, Kachelgröße und Zoomstufen sind identisch, alle Clients und Formeln des DTM gelten unverändert.



## Kennwerte



## Live-Karte



Der Klick dekodiert das Pixel der Kachel (Formel unter [Terrain-RGB](/de/reference/terrain-rgb/)).

## Einbinden

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/dsm.json", tileSize: 512, encoding: "mapbox" } },
        terrain: { source: "dsm", exaggeration: 1.2 },
        layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
      }
    });
    ```
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://https://tiles.mapx.at/relief-dsm-terrainrgb-20250915-d19cb434.pmtiles"` (Protokoll wie im [Schnellstart](/de/start/)).
    **Leaflet**

    ```js
    L.tileLayer("https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp", {
      tileSize: 512, zoomOffset: -1, crossOrigin: true,
      attribution: "BEV, CC BY 4.0"
    }).addTo(map);
    ```
    **QGIS**

    XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp`. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/dsm.json
    curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/dsm/14/8860/5730.webp
    ```
  
Höhe per HTTP: [Höhe und Profil](/de/api/elevation/) mit `model=dsm`.

## Quelle und Aktualität

Das DSM entsteht aus dem ALS-DOM 1 m des BEV (Airborne Laserscanning, Oberfläche). Der Stichtag steht in den Kennwerten und im Dateinamen; eine Datei ändert sich nie, ein neuer Stand ist eine neue Datei.

- Quelle: BEV ALS-DOM 1 m, Stichtag 15.09.2025.
- Gleiche Kodierung wie das DTM: Mapbox-Terrain-RGB mit 0,1 m Quantisierung, 512 px, WebP verlustfrei.

## Hinweise

- Quantisierung 0,1 m, maximal z16 (≈ 0,8 m pro Pixel bei 48° N), darüber zoomt der Client auf.
- Die Genauigkeit entspricht der amtlichen Quelle des BEV.
- Gebäude und Bewuchs entsprechen dem Zeitpunkt der Befliegung, nicht dem Stichtag.
- Außerhalb der Daten ist der Wert eine glatte Fortsetzung, keine Messung (wie beim DTM).
- Gebäude- und Bewuchshöhe (nDSM) ergeben sich im Client aus DSM minus DTM.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

{{< attribution ts="dsm" >}}

Mehr: [Lizenz](/de/reference/attribution/).


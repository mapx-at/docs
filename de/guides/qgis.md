# mapx-Daten in QGIS einbinden

Source: https://docs.mapx.at/de/guides/qgis/

> DTM, DSM, Orthophoto und Vektorkacheln in QGIS einbinden: XYZ-Verbindung und Vektorkachel-Verbindung.

**Ziel:** Orthophoto, Höhenmodelle und Vektordaten von mapx als Layer in QGIS verwenden, ohne Download, mit kostenlosem Token (an die Layer-URLs `?access_token=DEIN_TOKEN` hängen, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Auf dem eigenen Rechner einen geheimen Token (`mx_sk_…`) verwenden: ein öffentlicher Token gilt nur für seine erlaubten Origins, und QGIS ist keine Webseite.

**Voraussetzungen:** QGIS 3.x.

## Orthophoto als XYZ-Layer

1. Im Browser-Panel Rechtsklick auf **XYZ Tiles**, **Neue Verbindung**.
2. Name `mapx Orthophoto`, URL:

   ```
   https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN
   ```

3. **Min. Zoom** 5, **Max. Zoom** 18, **Kachelauflösung** 512 (Mapbox-Stil, 512 px). Bei **Referer** und **Authentifizierung** nichts eintragen; der Token in der URL genügt.
4. Mit OK speichern, die Verbindung per Doppelklick in die Karte laden.

Das Projekt-KBS darf beliebig sein, QGIS rechnet die Web-Mercator-Kacheln um.

## DTM und DSM

Die Höhenkacheln sind [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md): die Höhe steckt in den Farbkanälen. Dieselbe Verbindung wie oben mit der URL

```
https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN
```

(Max. Zoom 16; für das Oberflächenmodell `dsm` statt `terrain-rgb`). Der Layer zeigt zunächst die kodierte Farbe. Die Höhe ergibt sich im **Raster-Rechner** aus den drei Bändern:

```
-10000 + ("Layer@1" * 65536 + "Layer@2" * 256 + "Layer@3") * 0.1
```

(`Layer` durch den Namen des Layers ersetzen). Das Ergebnis ist die Höhe in Metern; darauf lassen sich Hangneigung, Exposition und Schummerung berechnen.

## Vektorkacheln

1. Browser-Panel, Rechtsklick auf **Vector Tiles**, **Neue generische Verbindung**.
2. Name `mapx Basiskarte`, URL:

   ```
   https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN
   ```

3. **Min. Zoom** 0, **Max. Zoom** 16 eintragen, OK.
4. Verbindung in die Karte ziehen. QGIS zeigt zunächst einen Standardstil; das Aussehen lässt sich im Layer-Stileditor oder mit einem eigenen Stil ändern.

Dieselbe Vorgehensweise gilt für `contours` (Max. 16, Min. 10), `orthophoto-flight-blocks` (Zoom 6 bis 17). Alle Tilesets: [Kacheln](https://docs.mapx.at/de/api/tiles.md).

## Nächste Schritte

- Höhenmodell verstehen: [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md).
- Quellenangabe in Karten und Berichten: [Attribution](https://docs.mapx.at/de/reference/attribution.md).

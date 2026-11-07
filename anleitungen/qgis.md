# mapx-Daten in QGIS

DTM, DSM, Orthophoto und Vektorkacheln in QGIS einbinden: XYZ-Verbindung, Vektorkachel-Verbindung und PMTiles direkt.

**Ziel:** Orthophoto, Höhenmodelle und Vektordaten von mapx als Layer in QGIS verwenden, ohne Download und ohne Schlüssel.

**Voraussetzungen:** QGIS 3.x (für PMTiles direkt ab 3.32).

## Orthophoto als XYZ-Layer

<Steps>

1. Im Browser-Panel Rechtsklick auf **XYZ Tiles**, **Neue Verbindung**.
2. Name `mapx Orthophoto`, URL:

   ```
   https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp
   ```

3. **Min. Zoom** 5, **Max. Zoom** 18, **Kachelauflösung** 512 (Mapbox-Stil, 512 px). Bei **Referer** und **Authentifizierung** nichts eintragen.
4. Mit OK speichern, die Verbindung per Doppelklick in die Karte laden.

</Steps>

Das Projekt-KBS darf beliebig sein, QGIS rechnet die Web-Mercator-Kacheln um.

## DTM und DSM

Die Höhenkacheln sind [Terrain-RGB](/konzepte/terrain-rgb/): die Höhe steckt in den Farbkanälen. Dieselbe Verbindung wie oben mit der URL

```
https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp
```

(Max. Zoom 16; für das Oberflächenmodell `dsm` statt `dtm`). Der Layer zeigt zunächst die kodierte Farbe. Die Höhe erhalten Sie im **Raster-Rechner** aus den drei Bändern:

```
-10000 + ("Layer@1" * 65536 + "Layer@2" * 256 + "Layer@3") * 0.1
```

(`Layer` durch den Namen Ihres Layers ersetzen). Das Ergebnis ist die Höhe in Metern; darauf lassen sich Hangneigung, Exposition und Schummerung berechnen.

## Vektorkacheln

<Steps>

1. Browser-Panel, Rechtsklick auf **Vector Tiles**, **Neue generische Verbindung**.
2. Name `mapx Basiskarte`, URL:

   ```
   https://tiles.mapx.at/v1/mapx-basis/{z}/{x}/{y}.pbf
   ```

3. **Min. Zoom** 0, **Max. Zoom** 16 eintragen, OK.
4. Verbindung in die Karte ziehen. QGIS zeigt zunächst einen Standardstil; das Aussehen ändern Sie im Layer-Stileditor oder mit einem eigenen Stil.

</Steps>

Dieselbe Vorgehensweise gilt für `hoehenlinien` (Max. 16, Min. 10), `mapx-kataster` (Zoom 16) und `dop-flugblock` (Zoom 6 bis 17). Alle Tilesets: [Kacheln](/api/kacheln/).

## PMTiles direkt (QGIS ab 3.32)

Vektor-Tilesets lassen sich ab QGIS 3.32 auch direkt als PMTiles-Datei öffnen, lokal oder per HTTP. Der Dateiname steht mit Stichtag auf der jeweiligen [Produktseite](/daten/) unter Kennwerte.

<Steps>

1. Browser-Panel, Rechtsklick auf **Vector Tiles**, **Neue generische Verbindung**.
2. **Diensttyp** auf PMTiles stellen (alternativ eine lokale Datei wählen) und als URL die Datei eintragen: `https://tiles.mapx.at/<datei>.pmtiles`.
3. Mit OK speichern und in die Karte laden. QGIS liest per HTTP-Range nur die benötigten Kacheln.

</Steps>

Bild- und Höhenkacheln (Orthophoto, DTM, DSM) binden Sie über die XYZ-Verbindung oben ein.

> Bietet Ihre QGIS-Version den PMTiles-Diensttyp nicht an, nehmen Sie die Vektorkachel-Verbindung oben; sie liefert dieselben Kacheln.

## Nächste Schritte

- Höhenmodell verstehen: [Terrain-RGB](/konzepte/terrain-rgb/).
- Daten lokal ablegen: [PMTiles selbst hosten](/anleitungen/pmtiles-selbst-hosten/).
- Quellenangabe in Karten und Berichten: [Attribution](/anleitungen/attribution/).
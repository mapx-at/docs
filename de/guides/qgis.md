# mapx-Daten in QGIS

DTM, DSM, Orthophoto und Vektorkacheln in QGIS einbinden: XYZ-Verbindung, Vektorkachel-Verbindung und PMTiles direkt.

**Ziel:** Orthophoto, Höhenmodelle und Vektordaten von mapx als Layer in QGIS verwenden, ohne Download, mit kostenlosem Token (an die Layer-URLs `?access_token=DEIN_TOKEN` hängen, siehe [Zugang und Limits](/de/start/access-and-limits/)). QGIS sendet keine Webseiten-Origin; dafür einen geheimen Token (`mx_sk_…`) verwenden und nur auf dem eigenen Rechner ablegen.

**Voraussetzungen:** QGIS 3.x (für PMTiles direkt ab 3.32).

## Orthophoto als XYZ-Layer

<Steps>

1. Im Browser-Panel Rechtsklick auf **XYZ Tiles**, **Neue Verbindung**.
2. Name `mapx Orthophoto`, URL:

   ```
   https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN
   ```

3. **Min. Zoom** 5, **Max. Zoom** 18, **Kachelauflösung** 512 (Mapbox-Stil, 512 px). Bei **Referer** und **Authentifizierung** nichts eintragen.
4. Mit OK speichern, die Verbindung per Doppelklick in die Karte laden.

</Steps>

Das Projekt-KBS darf beliebig sein, QGIS rechnet die Web-Mercator-Kacheln um.

## DTM und DSM

Die Höhenkacheln sind [Terrain-RGB](/de/reference/terrain-rgb/): die Höhe steckt in den Farbkanälen. Dieselbe Verbindung wie oben mit der URL

```
https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN
```

(Max. Zoom 16; für das Oberflächenmodell `dsm` statt `dtm`). Der Layer zeigt zunächst die kodierte Farbe. Die Höhe ergibt sich im **Raster-Rechner** aus den drei Bändern:

```
-10000 + ("Layer@1" * 65536 + "Layer@2" * 256 + "Layer@3") * 0.1
```

(`Layer` durch den Namen des Layers ersetzen). Das Ergebnis ist die Höhe in Metern; darauf lassen sich Hangneigung, Exposition und Schummerung berechnen.

## Vektorkacheln

<Steps>

1. Browser-Panel, Rechtsklick auf **Vector Tiles**, **Neue generische Verbindung**.
2. Name `mapx Basiskarte`, URL:

   ```
   https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN
   ```

3. **Min. Zoom** 0, **Max. Zoom** 16 eintragen, OK.
4. Verbindung in die Karte ziehen. QGIS zeigt zunächst einen Standardstil; das Aussehen lässt sich im Layer-Stileditor oder mit einem eigenen Stil ändern.

</Steps>

Dieselbe Vorgehensweise gilt für `contours` (Max. 16, Min. 10), `cadastre` (Zoom 16) und `orthophoto-flight-blocks` (Zoom 6 bis 17). Alle Tilesets: [Kacheln](/de/api/tiles/).

## PMTiles direkt (QGIS ab 3.32)

Vektor-Tilesets lassen sich ab QGIS 3.32 auch direkt als PMTiles-Datei öffnen, lokal oder per HTTP. Der Dateiname steht mit Stichtag auf der jeweiligen [Produktseite](/de/tilesets/) unter Kennwerte.

<Steps>

1. Browser-Panel, Rechtsklick auf **Vector Tiles**, **Neue generische Verbindung**.
2. **Diensttyp** auf PMTiles stellen (alternativ eine lokale Datei wählen) und als URL die Datei eintragen: `https://tiles.mapx.at/<datei>.pmtiles?access_token=DEIN_TOKEN`.
3. Mit OK speichern und in die Karte laden. QGIS liest per HTTP-Range nur die benötigten Kacheln.

</Steps>

Bild- und Höhenkacheln (Orthophoto, DTM, DSM) lassen sich über die XYZ-Verbindung oben einbinden.

> Bietet die QGIS-Version den PMTiles-Diensttyp nicht an, die Vektorkachel-Verbindung oben verwenden; sie liefert dieselben Kacheln.

## Nächste Schritte

- Höhenmodell verstehen: [Terrain-RGB](/de/reference/terrain-rgb/).
- Daten lokal ablegen: [PMTiles selbst hosten](/de/guides/self-host-pmtiles/).
- Quellenangabe in Karten und Berichten: [Attribution](/de/reference/attribution/).
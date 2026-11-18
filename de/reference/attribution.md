# Quellenangabe und Lizenz (CC BY 4.0)

Source: https://docs.mapx.at/de/reference/attribution/

> Welche Quelle bei mapx-Karten und -Daten unter CC BY 4.0 zu nennen ist: fertiger Text je Tileset und API, für MapLibre, Druck und Bilder.

Die amtlichen Daten in mapx stehen unter [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Ausnahme sind die Sentinel-2-Kacheln: Sie enthalten veränderte Copernicus-Sentinel-Daten und folgen den Copernicus-Bedingungen, die Quellenangabe steht im TileJSON. Die kommerzielle Nutzung ist erlaubt. Die Quelle und die Lizenz müssen genannt werden, Änderungen dürfen nicht als amtlich dargestellt werden. Die Quellenangabe muss für die Nutzer der Karte sichtbar sein, zum Beispiel in der Ecke, nicht nur im Impressum.

## In einer MapLibre-Karte

MapLibre zeigt die Attribution automatisch an, wenn ein Style oder ein TileJSON geladen wird (unten rechts, Symbol "i"). Es ist nichts zu ergänzen, solange sie nicht ausgeblendet wird. Bei `attributionControl: false` oder bei `tiles: [...]` ohne TileJSON den Text selbst ergänzen:

```js
{ type: "raster", tiles: ["https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN"], tileSize: 512,
  maxzoom: 18, attribution: "Orthophoto: BEV (CC BY 4.0)" }
```

## In Druck, Berichten und Static Images

Eine sichtbare Zeile neben das Bild setzen, in lesbarer Größe, zum Beispiel `Quelle: BEV – ALS-DTM 1 m (2025-09-15), CC BY 4.0`. Bei mehreren Quellen alle nennen, getrennt durch Strichpunkte.

## Text je Tileset

| Tileset | Text |
|---|---|
| `streets` | Datenquellen: BEV (DKM, Verwaltungsgrenzen, Adressregister, Geographische Namen), GIP.at, BMLUK (GGN), Umweltbundesamt; CC BY 4.0 |
| `cadastre` | Datenquelle: BEV – DKM, CC BY 4.0 |
| `contours` | Höhenlinien: mapx.at, Datenquelle BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `terrain-rgb` | Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `dsm` | Relief: BEV – ALS-DOM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `orthophoto` | Orthophoto: BEV (CC BY 4.0) |
| `orthophoto-flight-blocks` | BEV, CC BY 4.0 |

## Text je API

JSON-Antworten mit Daten tragen ein `attribution`-Feld auf oberster Ebene. Es ist anzuzeigen, wenn die Daten dargestellt oder weitergegeben werden.

| API | Quelle |
|---|---|
| Geocoding, Search Box, Address | BEV (Adressregister, Geographische Namen, Verwaltungsgrenzen, Kataster), CC BY 4.0 |
| Directions, Isochrone, Matrix, Matching | GIP.at, CC BY 4.0 |
| Maut, Verkehr | ASFINAG / EVIS.AT, CC BY 4.0, Straßennetz GIP.at |
| Transit | ÖBB-Personenverkehr AG (CC BY 4.0), Datenquelle Mobilitätsverbünde Österreich OG |
| Wetter | GeoSphere Austria, CC BY 4.0 |
| Klima, `properties.climate` des Grundstücks-Dossiers | GeoSphere Austria (APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE), CC BY 4.0 |
| Parcel, Tilequery (Kataster) | BEV – DKM, CC BY 4.0 |
| Höhe | BEV – ALS-DTM / ALS-DOM 1 m, CC BY 4.0 |

Die Quellenangabe steht auch in den Metadaten von TileJSON und PMTiles.

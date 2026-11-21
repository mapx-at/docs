# Kachelschema: Web-Mercator, z/x/y

Source: https://docs.mapx.at/de/reference/tile-scheme/

> Kachelschema der mapx-Kacheln: Web-Mercator, z/x/y-Adressierung, 512-Pixel-Kacheln, Auflösung je Zoomstufe und Overzoom.

Alle mapx-Kacheln folgen dem gängigen Schema von Web-Karten. Wer MapLibre, Leaflet, OpenLayers oder QGIS nutzt, muss dazu nichts einstellen; für eigene Abfragen und Berechnungen steht hier das Wichtigste.

## Projektion und Adresse

Die Welt wird in **Web-Mercator** (EPSG:3857) auf ein Quadrat abgebildet und in Zoomstufen unterteilt. Auf Stufe `z` hat das Quadrat `2^z × 2^z` Kacheln. Eine Kachel wird mit `z/x/y` adressiert: `x` zählt von West nach Ost, `y` von **Nord nach Süd** (XYZ-Schema, nicht TMS), beide beginnen bei 0.

Kachel für einen Punkt `lon, lat` (Grad) auf Stufe `z`:

```js
function kachel(lon, lat, z) {
  const n = 2 ** z;
  const x = Math.floor(((lon + 180) / 360) * n);
  const r = (lat * Math.PI) / 180;
  const y = Math.floor(((1 - Math.log(Math.tan(r) + 1 / Math.cos(r)) / Math.PI) / 2) * n);
  return { z, x, y };
}
kachel(12.694, 47.074, 12);   // { z: 12, x: 2192, y: 1439 }
```

Eine Kachel liefert der Endpunkt `https://tiles.mapx.at/v1/tiles/{tileset}/{z}/{x}/{y}.{ext}`, siehe [Kacheln](https://docs.mapx.at/de/api/tiles.md).

## 512-Pixel-Kacheln

Raster-Kacheln von mapx (Höhenmodelle, Orthofoto) haben **512 × 512 Pixel**, nicht 256. Dadurch deckt eine 512er-Kachel dasselbe Gebiet ab wie vier 256er-Kacheln eine Stufe tiefer. Clients brauchen dafür die richtige Einstellung:

- MapLibre: `tileSize: 512` (bei TileJSON automatisch).
- Leaflet: `tileSize: 512, zoomOffset: -1`.
- OpenLayers: `tileSize: 512` an der Quelle.
- QGIS: Kachelauflösung 512.

Vergisst man das, wirkt die Karte um eine Zoomstufe versetzt und unscharf oder zu fein.

## Auflösung je Zoomstufe

Die Auflösung hängt von der Breite ab. Die Tabelle gilt für 47,5° N, die Mitte Österreichs:

| Zoom | m pro Pixel | Kantenlänge Kachel (km) | Anmerkung |
|---|---|---|---|
| z5 | 1.652 | 846 |  |
| z6 | 826 | 423 |  |
| z7 | 413 | 212 |  |
| z8 | 207 | 106 |  |
| z9 | 103 | 53 |  |
| z10 | 52 | 26 |  |
| z11 | 26 | 13 |  |
| z12 | 13 | 6,61 |  |
| z13 | 6,46 | 3,30 |  |
| z14 | 3,23 | 1,65 |  |
| z15 | 1,61 | 0,83 |  |
| z16 | 0,81 | 0,41 | höchste gespeicherte Stufe (≈ 1-m-Ursprung) |
| z17 | 0,40 | 0,21 | Overzoom (Client) |

Eine Stufe tiefer halbiert die Pixelgröße. Die Höhenmodelle liegen bis z16 vor, das Orthofoto bis z18.

## Overzoom

Ist eine Stufe höher als die gespeicherte, **vergrößert der Client** die Kacheln der höchsten Stufe (Overzoom). Das Bild wird dabei nicht schärfer, die Karte bleibt aber bedienbar. In MapLibre dazu `maxzoom` an der Quelle setzen auf die höchste vorhandene Stufe (aus TileJSON automatisch), die Karte selbst darf weiter zoomen. Abfragen außerhalb des Zoombereichs liefern `404`, Kacheln innerhalb des Zoombereichs, aber außerhalb Österreichs `204` ohne Inhalt, siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

> Vektorkacheln werden beim Overzoom nicht neu gezeichnet, sondern die Geometrie der höchsten Stufe wird gestreckt.

## Weiter

- Höhenwerte in Kacheln: [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md).
- Koordinaten und Projektionen: [Koordinaten](https://docs.mapx.at/de/reference/coordinates.md).

# Glossar

Begriffe aus der mapx-Dokumentation.

**Annotation.** Zusätzliche Werte je Routensegment, zum Beispiel Geschwindigkeit oder Höhe. Angefordert mit `annotations`.

**Katastralgemeinde (KG).** Einheit des österreichischen Grundbuchs. Eine Grundstücksnummer ist nur innerhalb einer KG eindeutig.

**Contours.** Höhenlinien, Linien gleicher Höhe. Tileset `contours`.

**DEM.** Digitales Höhenmodell, hier als Terrain-RGB-Tiles gespeichert. Siehe [Terrain-RGB](/de/reference/terrain-rgb/).

**DSM / DTM.** Digitales Oberflächenmodell (mit Gebäuden und Bäumen) und digitales Geländemodell (nackter Boden).

**Geocoding.** Eine Adresse oder einen Ortsnamen in Koordinaten umwandeln. Reverse Geocoding macht das Gegenteil.

**Glyphs.** Schriftdaten, die MapLibre lädt, um Beschriftungen zu zeichnen. Siehe [Styles API](/de/api/styles/).

**Isochrone.** Ein Gebiet, das von einem Punkt aus innerhalb einer bestimmten Zeit oder Distanz erreichbar ist.

**Orthophoto.** Entzerrte Luftaufnahme.

**PMTiles.** Einzeldatei-Archiv von Tiles, gelesen mit HTTP-Range-Requests. Jedes Tileset ist als PMTiles verfügbar.

**Profil.** Die Art des Verkehrsteilnehmers beim Routing: `driving`, `driving-truck`, `cycling`, `walking`.

**Sprite.** Bildatlas mit den Icons eines Styles.

**Style.** Ein JSON-Dokument, das der Karte sagt, welche Daten sie wie zeichnet. mapx bietet `streets`, `satellite` und `outdoors`.

**Tile.** Ein quadratisches Kartenstück, adressiert als `{z}/{x}/{y}`. Siehe [Tile-Schema](/de/reference/tile-scheme/).

**TileJSON.** Eine kleine JSON-Beschreibung eines Tilesets: URL-Vorlage, Zoombereich, Grenzen, Attribution.

**Tileset.** Eine benannte Sammlung von Tiles, zum Beispiel `streets` oder `cadastre`. Siehe [Tilesets](/de/tilesets/).
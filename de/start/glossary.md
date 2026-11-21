# Glossar: mapx- und GIS-Begriffe

Source: https://docs.mapx.at/de/start/glossary/

> Glossar der Begriffe aus der mapx-Dokumentation: Annotation, Höhenlinien, Isochronen, PMTiles, Terrain-RGB und weitere.

**Annotation.** Zusätzliche Werte je Routensegment, zum Beispiel Geschwindigkeit oder Höhe. Angefordert mit `annotations`.

**Contours.** Höhenlinien, Linien gleicher Höhe. Kachelsatz `contours`.

**DEM.** Digitales Höhenmodell, hier als Terrain-RGB-Kacheln gespeichert. Siehe [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md).

**DSM / DTM.** Digitales Oberflächenmodell (mit Gebäuden und Bäumen) und digitales Geländemodell (nackter Boden).

**Geocoding.** Eine Adresse oder einen Ortsnamen in Koordinaten umwandeln. Reverse Geocoding macht das Gegenteil.

**Glyphs.** Schriftdaten, die MapLibre lädt, um Beschriftungen zu zeichnen. Siehe [Styles API](https://docs.mapx.at/de/api/styles.md).

**Isochrone.** Ein Gebiet, das von einem Punkt aus innerhalb einer bestimmten Zeit oder Distanz erreichbar ist.

**Orthofoto.** Entzerrte Luftaufnahme.

**PMTiles.** Einzeldatei-Archiv von Kacheln, gelesen mit HTTP-Range-Requests. Die mapx-Stile lesen ihre Kacheln aus PMTiles; dafür muss die Seite das Protokoll `pmtiles://` registrieren (`maplibregl.addProtocol`, siehe [Erste Karte](https://docs.mapx.at/de/start/first-map.md)). Zugriff für eigene Anwendungen sonst über TileJSON oder `{z}/{x}/{y}`.

**Profil.** Die Art des Verkehrsteilnehmers beim Routing: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

**Sprite.** Bildatlas mit den Icons eines Stils.

**Stil.** Ein JSON-Dokument, das der Karte sagt, welche Daten sie wie zeichnet. mapx bietet unter anderem `streets`, `satellite` und `outdoors`; alle Stile stehen in der [Stil-Referenz](https://docs.mapx.at/de/reference/style-spec.md).

**Kachel.** Ein quadratisches Kartenstück, adressiert als `{z}/{x}/{y}`. Siehe [Kachelschema](https://docs.mapx.at/de/reference/tile-scheme.md).

**TileJSON.** Eine kleine JSON-Beschreibung eines Kachelsatzes: URL-Vorlage, Zoombereich, Grenzen, Quellenangabe.

**Kachelsatz.** Eine benannte Sammlung von Kacheln, zum Beispiel `streets` oder `contours`. Siehe [Kachelsätze](https://docs.mapx.at/de/tilesets/index.md).

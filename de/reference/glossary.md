# Glossar: mapx- und GIS-Begriffe

Source: https://docs.mapx.at/de/reference/glossary/

> Glossar der Begriffe aus der mapx-Dokumentation: Annotation, Höhenlinien, Isochronen, PMTiles, Terrain-RGB und weitere.

**Annotation.** Zusätzliche Werte je Routensegment, zum Beispiel Geschwindigkeit oder Höhe. Angefordert mit `annotations`.

**Attribution.** Pflichtangabe der Datenquellen (CC BY 4.0). Jede JSON-Antwort trägt sie im Feld `attribution`, Form `mapx.at | <Quelle> (<Lizenz>); …`. Siehe [Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md).

**Closures (Sperren).** Aktive Vollsperren aus den Verkehrsmeldungen (EVIS/ASFINAG), die Directions, Matrix, Isochrone und Optimierung automatisch umfahren. Der Block `closures` (`applied`, `count`, `binding`, `binding_count`, `fetched`) sagt, welche Sperren galten. Siehe [Navigation](https://docs.mapx.at/de/guides/navigation.md).

**Contours.** Höhenlinien, Linien gleicher Höhe. Tileset `contours`.

**DEM.** Digitales Höhenmodell, hier als Terrain-RGB-Kacheln gespeichert. Siehe [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md).

**DSM / DTM.** Digitales Oberflächenmodell (mit Gebäuden und Bäumen) und digitales Geländemodell (nackter Boden).

**Geocoding.** Eine Adresse oder einen Ortsnamen in Koordinaten umwandeln. Reverse Geocoding macht das Gegenteil.

**Glyphs.** Schriftdaten, die MapLibre lädt, um Beschriftungen zu zeichnen. Siehe [Styles API](https://docs.mapx.at/de/api/styles.md).

**Isochrone.** Ein Gebiet, das von einem Punkt aus innerhalb einer bestimmten Zeit oder Distanz erreichbar ist.

**Kachel.** Ein quadratisches Kartenstück, adressiert als `{z}/{x}/{y}`. Siehe [Kachelschema](https://docs.mapx.at/de/reference/tile-scheme.md).

**Map Matching.** Eine aufgezeichnete GPS-Spur dem Straßengraphen zuordnen. Siehe [Map Matching API](https://docs.mapx.at/de/api/map-matching.md).

**Matrix.** Dauer und Distanz zwischen vielen Start- und Zielpunkten in einer Anfrage. Siehe [Matrix API](https://docs.mapx.at/de/api/matrix.md).

**`modes` (Verkehrsmittel).** Filter und Feld der ÖV-Endpunkte, zum Beispiel `rail`, `bus`, `tram`, `subway`. Siehe [ÖV API](https://docs.mapx.at/de/api/transit.md).

**Origin.** Herkunft einer Browser-Anfrage (`https://example.com`). Öffentliche Tokens (`mx_pk_`) funktionieren nur von erlaubten Origins. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

**Orthofoto.** Entzerrte Luftaufnahme.

**Place.** Das gemeinsame Antwortformat aller Such-Endpunkte (Geocoding, Search Box, Autofill): GeoJSON-Feature mit `properties.type`, `name`, `place_formatted`, `relevance`, `accuracy` und `address`. Siehe [Geocoding API](https://docs.mapx.at/de/api/geocoding.md).

**PMTiles.** Einzeldatei-Archiv von Kacheln, gelesen mit HTTP-Range-Requests. Die mapx-Stile lesen ihre Kacheln aus PMTiles; dafür muss die Seite das Protokoll `pmtiles://` registrieren (`maplibregl.addProtocol`, siehe [Erste Karte](https://docs.mapx.at/de/start/first-map.md)). Zugriff für eigene Anwendungen sonst über TileJSON oder `{z}/{x}/{y}`.

**POI.** Point of Interest, ein Ort wie Tankstelle, Museum oder Bahnhof. Suchbar nach Kategorie über die [Search Box API](https://docs.mapx.at/de/api/search-box.md), auf der Karte in der Ebene `poi_p` von `streets`.

**Problem Details.** Fehlerformat nach RFC 9457: JSON mit `type`, `title`, `status`, `detail` und dem maschinenlesbaren `code` (zum Beispiel `unknown-parameter`). `detail` ist immer englisch, auswerten über `code`. Siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

**Profil.** Die Art des Verkehrsteilnehmers beim Routing: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`, bei der Isochrone zusätzlich `transit`.

**Sprite.** Bildatlas mit den Icons eines Stils.

**Stil.** Ein JSON-Dokument, das der Karte sagt, welche Daten sie wie zeichnet. mapx bietet unter anderem `streets`, `satellite` und `outdoors`; alle Stile stehen in der [Stil-Referenz](https://docs.mapx.at/de/reference/style-spec.md).

**TileJSON.** Eine kleine JSON-Beschreibung eines Tilesets: URL-Vorlage, Zoombereich, Grenzen, Quellenangabe.

**Tileset.** Eine benannte Sammlung von Kacheln, zum Beispiel `streets` oder `contours`. Siehe [Tilesets](https://docs.mapx.at/de/tilesets/index.md).

**Token.** Zugangsschlüssel zur API: `mx_pk_…` öffentlich (Webseiten und Apps, an Origins gebunden), `mx_sk_…` geheim (Server), `mx_mcp_…` für KI-Assistenten über den MCP-Server. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

**`transit`.** ÖV-Profil der Isochrone: zu Fuß plus öffentlicher Verkehr ab `depart_at`. Verbindungen mit ÖV liefert die [ÖV API](https://docs.mapx.at/de/api/transit.md).

**Zone.** Gruppe von Endpunkten mit gemeinsamem Rate-Limit, zum Beispiel `search`, `route`, `heavy`, `tiles`. Jede Antwort nennt sie im Header `RateLimit-Policy`. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

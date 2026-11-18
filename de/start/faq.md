# FAQ: Token, Lizenz, Offline-Nutzung

Source: https://docs.mapx.at/de/start/faq/

> Antworten zu mapx: Token nötig, Kosten, kommerzielle Nutzung unter CC BY 4.0, Aktualität der Daten, Offline-Karten, Daten außerhalb Österreichs, Fehler 401 und 403.

## Brauche ich einen Token oder ein Konto?

Ja, ein kostenloses Konto und einen API-Token. Unter [account.mapx.at](https://account.mapx.at/) registrieren, Token anlegen und als `Authorization: Bearer` oder `access_token` mitschicken. CORS ist offen. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

## Kann ich mapx mit Claude, ChatGPT oder Cursor nutzen?

Ja, über den [MCP-Server](https://docs.mapx.at/de/guides/mcp.md): 23 lesende Werkzeuge für Österreich-Daten. Lege einen Token vom Typ „KI-Assistent (MCP)“ an und trage `https://mcp.mapx.at/mcp` in Claude Code, Cursor, VS Code oder Gemini CLI ein. Connectoren in claude.ai und ChatGPT folgen später.

## Was kostet es?

Nichts. Rate Limits gelten pro Token und Zone.

## Ist kommerzielle Nutzung erlaubt?

Ja. Die Daten stehen unter [CC BY 4.0](https://docs.mapx.at/de/reference/attribution.md): die Quelle nennen, alles andere ist frei.

## Wie aktuell sind die Daten?

Jedes Tileset hat ein Quelldatum. Es steht auf seiner Seite unter [Tilesets](https://docs.mapx.at/de/tilesets/index.md) und im Feld `stand` von `GET /v1/tiles`.

## Ändert sich eine URL bei neuen Daten?

API-URLs (`/v1/tiles/streets/...`, TileJSON) bleiben gleich. PMTiles-Dateinamen enthalten das Datum, alte Dateien bleiben also gültig. Über das TileJSON ist immer die aktuelle Datei erreichbar.

## Lassen sich die Karten offline nutzen?

Ja. Eine PMTiles-Datei herunterladen und selbst ausliefern: [PMTiles selbst hosten](https://docs.mapx.at/de/guides/self-host-pmtiles.md), [QGIS](https://docs.mapx.at/de/guides/qgis.md).

## Gibt es Daten außerhalb Österreichs?

Nur Österreich. Orthophoto und Basiskarte reichen an der Grenze ein Stück in die Nachbarländer.

## Was ist der Unterschied zwischen DTM und DSM?

Das DTM ist der nackte Boden, das DSM enthält Bäume und Gebäude. Das DTM für Steigungen und Profile verwenden, das DSM für Sichtbarkeit und Gebäudehöhen. Siehe [Höhen](https://docs.mapx.at/de/guides/elevation.md).

## Warum kommt 404 für ein Tile?

Das Tile liegt außerhalb des Datengebiets oder Zoombereichs, die Endung passt nicht zum Typ (`pbf` Vektor, `webp` Raster) oder der Tileset-Name ist falsch. 404 als leeres Tile behandeln.

## Warum kommt 429?

Das Rate Limit der Zone wurde überschritten. `Retry-After` Sekunden warten und mit wachsender Pause wiederholen. Bei hoher Last [PMTiles](https://docs.mapx.at/de/guides/self-host-pmtiles.md) verwenden.

## Meine Punkte landen im falschen Land

Längen- und Breitengrad sind vertauscht. mapx und GeoJSON verwenden `lon,lat`. Siehe [Koordinaten](https://docs.mapx.at/de/reference/coordinates.md).

## Meine Karte ist unscharf

Raster-Tiles haben 512 px. Im Client `tileSize: 512` setzen. Siehe [Tile-Schema](https://docs.mapx.at/de/reference/tile-scheme.md).

## Ist der Kataster rechtsverbindlich?

Nein. Er zeigt Grundstücksgrenzen zur Orientierung und ersetzt weder einen Grundbuchauszug noch eine Vermessung.

## Welche alten Namen funktionieren noch?

Die früheren deutschen Style-, Tileset- und Kategorienamen funktionieren bis 2027-05-31. Siehe [Migration](https://docs.mapx.at/de/api/migration.md).

## Wo melde ich ein Problem?

Ein Issue in der [GitHub-Organisation mapx-at](https://github.com/mapx-at) eröffnen.

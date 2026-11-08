# FAQ

Kurze Antworten zu Keys, Lizenz, Aktualität, Offline-Nutzung und häufigen Fehlern.

## Brauche ich einen Key oder ein Konto?

Nein. Alle APIs und Tiles sind öffentlich, CORS ist offen. Siehe [Zugang und Limits](/de/start/access-and-limits/).

## Was kostet es?

Nichts. Rate Limits gelten pro IP-Adresse und Zone.

## Ist kommerzielle Nutzung erlaubt?

Ja. Die Daten stehen unter [CC BY 4.0](/de/reference/attribution/): die Quelle nennen, alles andere ist frei.

## Wie aktuell sind die Daten?

Jedes Tileset hat ein Quelldatum. Es steht auf seiner Seite unter [Tilesets](/de/tilesets/) und im Feld `stand` von `GET /v1/tiles`.

## Ändert sich eine URL bei neuen Daten?

API-URLs (`/v1/tiles/streets/...`, TileJSON) bleiben gleich. PMTiles-Dateinamen enthalten das Datum, alte Dateien bleiben also gültig. Über das TileJSON ist immer die aktuelle Datei erreichbar.

## Lassen sich die Karten offline nutzen?

Ja. Eine PMTiles-Datei herunterladen und selbst ausliefern: [PMTiles selbst hosten](/de/guides/self-host-pmtiles/), [QGIS](/de/guides/qgis/).

## Gibt es Daten außerhalb Österreichs?

Nur Österreich. Orthophoto und Basiskarte reichen an der Grenze ein Stück in die Nachbarländer.

## Was ist der Unterschied zwischen DTM und DSM?

Das DTM ist der nackte Boden, das DSM enthält Bäume und Gebäude. Das DTM für Steigungen und Profile verwenden, das DSM für Sichtbarkeit und Gebäudehöhen. Siehe [Höhen](/de/guides/elevation/).

## Warum kommt 404 für ein Tile?

Das Tile liegt außerhalb des Datengebiets oder Zoombereichs, die Endung passt nicht zum Typ (`pbf` Vektor, `webp` Raster) oder der Tileset-Name ist falsch. 404 als leeres Tile behandeln.

## Warum kommt 429?

Das Rate Limit der Zone wurde überschritten. `Retry-After` Sekunden warten und mit wachsender Pause wiederholen. Bei hoher Last [PMTiles](/de/guides/self-host-pmtiles/) verwenden.

## Meine Punkte landen im falschen Land

Längen- und Breitengrad sind vertauscht. mapx und GeoJSON verwenden `lon,lat`. Siehe [Koordinaten](/de/reference/coordinates/).

## Meine Karte ist unscharf

Raster-Tiles haben 512 px. Im Client `tileSize: 512` setzen. Siehe [Tile-Schema](/de/reference/tile-scheme/).

## Ist der Kataster rechtsverbindlich?

Nein. Er zeigt Grundstücksgrenzen zur Orientierung und ersetzt weder einen Grundbuchauszug noch eine Vermessung.

## Welche alten Namen funktionieren noch?

Die früheren deutschen Style-, Tileset- und Kategorienamen funktionieren bis 2027-05-31. Siehe [Migration](/de/api/migration/).

## Wo melde ich ein Problem?

Ein Issue in der [GitHub-Organisation mapx-at](https://github.com/mapx-at) eröffnen.
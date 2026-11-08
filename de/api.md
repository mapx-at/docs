# API-Referenz

Alle mapx-APIs: Endpunkte, Parameter, Beispiele und ein Live-Playground, generiert aus der OpenAPI-Beschreibung.

Alle APIs liegen unter `https://api.mapx.at/v1/` und brauchen einen kostenlosen API-Token ([Zugang und Limits](/de/start/access-and-limits/)). Die folgenden Seiten sind aus der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) generiert und passen immer zum laufenden Dienst. Jede Seite hat einen Playground, der die Live-API aufruft.

## Konventionen

- **Koordinaten** sind `lon,lat` (WGS84), mehrere getrennt durch `;`. Siehe [Koordinaten](/de/reference/coordinates/).
- **Fehler** sind `application/problem+json` (RFC 9457) mit stabilem `code`. Siehe [Fehler und Limits](/de/api/errors/).
- **Limits** gelten pro Token und Zone; `429` trägt `Retry-After`. Siehe [Zugang und Limits](/de/start/access-and-limits/).
- **Caching:** Tiles, Styles und TileJSON tragen `ETag`; mit `If-None-Match` kommt `304`.
- **Versionierung:** Die Version steht im Pfad. Felder ändern sich innerhalb einer Version nicht, neue kommen nur hinzu. Deprecations werden im [Changelog](/de/reference/changelog/) mit Stichtag angekündigt.

## APIs

                              
## Weitere Referenz

- [Tilesets](/de/tilesets/): Datenblätter aller Tilesets.
- [Style-Spezifikation](/de/reference/style-spec/), [Terrain-RGB](/de/reference/terrain-rgb/), [Tile-Schema](/de/reference/tile-scheme/).
- [Fehler und Limits](/de/api/errors/), [Migration](/de/api/migration/), [Changelog](/de/reference/changelog/), [Attribution](/de/reference/attribution/).
- [Status](https://api.mapx.at/health): Health-Check der API.
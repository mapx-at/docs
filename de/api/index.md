# API-Referenz


Alle APIs liegen unter `https://api.mapx.at/v1/` und brauchen einen kostenlosen API-Token ([Zugang und Limits](/de/start/access-and-limits/)). Die folgenden Seiten sind aus der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) generiert und passen immer zum laufenden Dienst. Jede Seite hat einen Playground, der die Live-API aufruft.

## Konventionen

- **Koordinaten** sind `lon,lat` (WGS84), mehrere getrennt durch `;`. Siehe [Koordinaten](/de/reference/coordinates/).
- **Fehler** sind `application/problem+json` (RFC 9457) mit stabilem `code`. Siehe [Fehler und Limits](/de/api/errors/).
- **Limits** gelten pro Token und Zone; `429` trägt `Retry-After`. Siehe [Zugang und Limits](/de/start/access-and-limits/).
- **Caching:** Tiles, Styles und TileJSON tragen `ETag`; mit `If-None-Match` kommt `304`.
- **Versionierung:** Die Version steht im Pfad. Felder ändern sich innerhalb einer Version nicht, neue kommen nur hinzu. Deprecations werden im [Changelog](/de/reference/changelog/) mit Stichtag angekündigt.

## APIs

  - [Tiles](/de/api/tiles/) Tileset-Liste, TileJSON, XYZ-Tiles.
  - [Styles](/de/api/styles/) Styles, Fonts, Sprites, Static Tiles.
  - [Geocoding](/de/api/geocoding/) Forward, Reverse und Batch.
  - [Search Box](/de/api/search-box/) Vorschläge, Retrieve, Kategorien.
  - [Address](/de/api/address/) Autofill und Validierung.
  - [Directions](/de/api/directions/) Routen, Isochrone, Matrix, Matching.
  - [Optimization](/de/api/optimization/) Optimierte Touren und Tourenplanung.
  - [Transit](/de/api/transit/) Verbindungen, Haltestellen, Abfahrten.
  - [Static Images](/de/api/static-images/) Kartenbilder mit Overlays.
  - [Tilequery und Parcel](/de/api/tilequery/) Features und Grundstücke an einem Punkt.
  - [Elevation](/de/api/elevation/) Höhen und Profile.
  - [Weather](/de/api/weather/) Beobachtungen, Vorhersage, Warnungen, Tiles.
  - [Air quality](/de/api/air-quality/) Index, Schadstoffe, Saharastaub.
  - [Climate](/de/api/climate/) Sonnenpotenzial, Normale, Schnee, Dürre.
  - [Traffic](/de/api/traffic/) Störungen, Tempolimits, Tiles.
## Weitere Referenz

- [Tilesets](/de/tilesets/): Datenblätter aller Tilesets.
- [Style-Spezifikation](/de/reference/style-spec/), [Terrain-RGB](/de/reference/terrain-rgb/), [Tile-Schema](/de/reference/tile-scheme/).
- [Fehler und Limits](/de/api/errors/), [Migration](/de/api/migration/), [Changelog](/de/reference/changelog/), [Attribution](/de/reference/attribution/).
- [Status](https://api.mapx.at/health): Health-Check der API.


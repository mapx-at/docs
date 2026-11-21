# API-Referenz: alle Endpunkte

Source: https://docs.mapx.at/de/api/

> Referenz aller mapx-APIs für Österreich mit Endpunkten, Parametern, Beispielen und Live-Playground, erzeugt aus der OpenAPI-Beschreibung.

Alle APIs liegen unter `https://api.mapx.at/v1/` und brauchen einen kostenlosen API-Token von [account.mapx.at](https://account.mapx.at/). Er geht als `Authorization: Bearer DEIN_TOKEN` oder `?access_token=DEIN_TOKEN` mit, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). `mx_pk_`-Tokens (öffentlich) funktionieren von erlaubten Websites und Android-Apps mit `GET`/`HEAD`, `mx_sk_`-Tokens (geheim) von Servern. Die folgenden Seiten sind aus der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) generiert (Stand des letzten Doku-Baus). Jede Seite hat einen Playground, der die Live-API aufruft.

## Konventionen

- **Koordinaten** sind `lon,lat` (WGS84), mehrere getrennt durch `;`. Siehe [Koordinaten](https://docs.mapx.at/de/reference/coordinates.md).
- **Fehler** sind `application/problem+json` (RFC 9457) mit stabilem `code`, auch `401`/`403` bei Token-Problemen. Siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).
- **Limits** gelten pro IP-Adresse und Zone, dazu ein Minutenlimit je Token; `429` trägt `Retry-After`. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).
- **Caching:** Kacheln, Stile und TileJSON tragen `ETag`; mit `If-None-Match` kommt `304`.
- **Versionierung:** Die Version steht im Pfad. Felder ändern sich innerhalb einer Version nicht, neue kommen nur hinzu.

## APIs

- [Tiles](https://docs.mapx.at/de/api/tiles.md): Tileset-Liste, TileJSON, XYZ-Tiles.
- [Styles](https://docs.mapx.at/de/api/styles.md): Styles, Fonts, Sprites, Static Tiles.
- [Geocoding](https://docs.mapx.at/de/api/geocoding.md): Forward, Reverse und Batch.
- [Search Box](https://docs.mapx.at/de/api/search-box.md): Vorschläge, Retrieve, Kategorien.
- [Address](https://docs.mapx.at/de/api/address.md): Autofill und Validierung.
- [Directions](https://docs.mapx.at/de/api/directions.md): Routen, Isochrone, Matrix, Matching.
- [Optimization](https://docs.mapx.at/de/api/optimization.md): Optimierte Touren und Tourenplanung.
- [Transit](https://docs.mapx.at/de/api/transit.md): Verbindungen, Haltestellen, Abfahrten.
- [Static Images](https://docs.mapx.at/de/api/static-images.md): Kartenbilder mit Overlays.
- [Tilequery](https://docs.mapx.at/de/api/tilequery.md): Features der Vektorkacheln an einem Punkt.
- [Elevation](https://docs.mapx.at/de/api/elevation.md): Höhen und Profile.
- [Weather](https://docs.mapx.at/de/api/weather.md): Beobachtungen, Vorhersage, Warnungen, Tiles.
- [Air quality](https://docs.mapx.at/de/api/air-quality.md): Index, Schadstoffe, Saharastaub.
- [Climate](https://docs.mapx.at/de/api/climate.md): Sonnenpotenzial, Normale, Schnee, Dürre.
- [Traffic](https://docs.mapx.at/de/api/traffic.md): Störungen, Tempolimits, Tiles.

## Weitere Referenz

- [Tilesets](https://docs.mapx.at/de/tilesets/index.md): Datenblätter aller Tilesets.
- [Style-Spezifikation](https://docs.mapx.at/de/reference/style-spec.md), [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md), [Tile-Schema](https://docs.mapx.at/de/reference/tile-scheme.md).
- [Fehler und Limits](https://docs.mapx.at/de/api/errors.md), [Attribution](https://docs.mapx.at/de/reference/attribution.md).
- [Status](https://api.mapx.at/health): Health-Check der API.

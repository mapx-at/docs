# API-Übersicht

Alle mapx-APIs: Kacheln, TileJSON und Höhe. Ohne Schlüssel, CORS offen, Fehler als RFC 9457.

Alle APIs folgen denselben Regeln:

- **Kein Schlüssel** – keine Anmeldung, kein Token, CORS offen für jede Website.
- **Version im Pfad** – `/v1/`; bestehende Felder ändern sich innerhalb einer Version nicht, neue kommen nur dazu.
- **Koordinaten** – immer `lon,lat` (WGS84), mehrere getrennt durch `;`. Mehr: [Koordinaten](/wissen/koordinaten/).
- **Fehler** – `application/problem+json` nach [RFC 9457](https://www.rfc-editor.org/rfc/rfc9457).
- **Limits** – je IP-Adresse, darüber `429` mit `Retry-After`.

## Karten

| API | Endpunkt | Zweck |
|---|---|---|
| [Kacheln](/api/kacheln/) | `GET tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}` | Raster-, Höhen- und Vektorkacheln |
| [PMTiles](/api/kacheln/#pmtiles-datei) | `GET tiles.mapx.at/<datei>.pmtiles` | ganzes Produkt als Datei, per HTTP-Range |
| [TileJSON](/api/tilejson/) | `GET tiles.mapx.at/v1/{tileset}.json` | Beschreibung eines Produkts für Kartenclients |

## Höhe

| API | Endpunkt | Zweck |
|---|---|---|
| [Höhe und Profil](/api/hoehe/) | `GET api.mapx.at/v1/elevation`, `/v1/elevation/profile` | Höhe für Punkte, Höhenprofil entlang einer Linie |

## Bilder

| API | Endpunkt | Zweck |
|---|---|---|
| [Static Images](/api/static/) | `GET api.mapx.at/v1/static/{style}/{view}/{size}` | Kartenbild als PNG oder WebP, mit Overlays |
| [SDK mapx-gl](/api/sdk/) | `map.mapx.at/sdk/v1/` | MapLibre-Aufsatz mit Suche, Stilwechsel, Profil |

## Stile, Suche und Routing

| API | Endpunkt | Zweck |
|---|---|---|
| [Styles](/api/styles/) | `GET api.mapx.at/v1/styles/{id}` | fertige MapLibre-Stile |
| [Geocoding](/api/geocoding/) | `GET api.mapx.at/v1/geocode`, `/v1/reverse` | Suche und Adresse zum Punkt |
| [Directions](/api/routing/#directions) | `GET api.mapx.at/v1/directions/{profile}/{coordinates}` | Route für Fuß, Rad, Auto |
| [Isochrone](/api/routing/#isochrone) | `GET api.mapx.at/v1/isochrone/{profile}/{coordinate}` | Erreichbarkeitsflächen |
| [Matrix](/api/routing/#matrix) | `GET api.mapx.at/v1/matrix/{profile}/{coordinates}` | Fahrzeiten zwischen vielen Orten |
| [Map Matching](/api/routing/#map-matching) | `GET api.mapx.at/v1/matching/{profile}/{coordinates}` | GPS-Spur auf das Wegenetz |

Alle Endpunkte laufen unter `https://api.mapx.at/v1/` ohne Zugangstoken. [Beispiele](/beispiele/) zeigen jeden Weg live.

## Betrieb

| Endpunkt | Zweck |
|---|---|
| `GET api.mapx.at/v1/openapi.json` | OpenAPI-Beschreibung aller Endpunkte |
| `GET api.mapx.at/health` | Lebenszeichen für Monitoring |

        
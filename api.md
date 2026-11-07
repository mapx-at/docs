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

## Wetter

| API | Endpunkt | Zweck |
|---|---|---|
| [Wetter](/api/weather/) | `GET api.mapx.at/v1/weather/current`, `/v1/weather/forecast`, `/v1/weather/tiles/{parameter}/{time}/{z}/{x}/{y}.png` | Messwerte, Stundenprognose, Wetterkacheln (GeoSphere Austria, CC BY 4.0) |

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
| [Search Box](/api/searchbox/) | `GET api.mapx.at/v1/searchbox/suggest`, `/retrieve/{id}`, `/category/{kategorie}` | Vorschläge, Details, Orte nach Kategorie |
| [Tilequery](/api/tilequery/) | `GET api.mapx.at/v1/tilequery/{tileset}/{lon},{lat}.json`, `/v1/parcel/{lon},{lat}.json` | Features, Grundstück und Höhe an einem Punkt |
| [Directions](/api/routing/#directions) | `GET api.mapx.at/v1/directions/{profile}/{coordinates}` | Route für Fuß, Rad, Auto |
| [Öffentlicher Verkehr](/api/transit/) | `GET api.mapx.at/v1/directions/transit/{coordinates}` | Verbindung mit ÖV, Fuß und Rad |
| [Isochrone](/api/routing/#isochrone) | `GET api.mapx.at/v1/isochrone/{profile}/{coordinate}` | Erreichbarkeitsflächen |
| [Matrix](/api/routing/#matrix) | `GET api.mapx.at/v1/matrix/{profile}/{coordinates}` | Fahrzeiten zwischen vielen Orten |
| [Map Matching](/api/routing/#map-matching) | `GET api.mapx.at/v1/matching/{profile}/{coordinates}` | GPS-Spur auf das Wegenetz |
| [Optimierte Reihenfolge](/api/routing/#optimierte-reihenfolge) | `GET api.mapx.at/v1/optimized-trips/{profile}/{coordinates}` | beste Reihenfolge für bis zu 25 Stopps |
| [Tourenplanung](/api/optimization/) | `POST api.mapx.at/v1/optimization` | Touren für mehrere Fahrzeuge mit Kapazität und Zeitfenstern |
| [Batch-Geocoding](/api/geocoding/#batch-geocoding) | `POST api.mapx.at/v1/geocode/batch` | bis zu 1000 Adressen in einem Aufruf |
| [Adress-Autofill](/api/autofill/) | `GET api.mapx.at/v1/autofill/suggest` | Adressen beim Tippen vervollständigen |
| [Adressvalidierung](/api/validation/) | `GET/POST api.mapx.at/v1/address/validate` | Adresse gegen das Adressregister prüfen |

Alle Endpunkte laufen unter `https://api.mapx.at/v1/` ohne Zugangstoken. [Beispiele](/beispiele/) zeigen jeden Weg live.

## Betrieb

| Endpunkt | Zweck |
|---|---|
| `GET api.mapx.at/v1/openapi.json` | OpenAPI-Beschreibung aller Endpunkte |
| `GET api.mapx.at/health` | Lebenszeichen für Monitoring |

        
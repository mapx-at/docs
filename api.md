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
| [Höhe und Profil](/api/hoehe/) | `GET api.mapx.at/v1/elevation` | Höhe für Punkte oder entlang einer Linie |

## Betrieb

| Endpunkt | Zweck |
|---|---|
| `GET api.mapx.at/v1/openapi.json` | OpenAPI-Beschreibung aller Endpunkte |
| `GET api.mapx.at/health` | Lebenszeichen für Monitoring |

<CardGrid>
  <LinkCard title="Kacheln" href="/api/kacheln/" description="XYZ und PMTiles für alle Produkte." />
  <LinkCard title="TileJSON" href="/api/tilejson/" description="Ein Produkt per URL einbinden." />
  <LinkCard title="Höhe und Profil" href="/api/hoehe/" description="Punkte und Profile, mit Testkonsole." />
  <LinkCard title="Fehler und Limits" href="/api/fehler/" description="Fehlerformat, Limits, Caching." />
</CardGrid>
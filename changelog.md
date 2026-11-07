# Changelog

Änderungen an Datenprodukten, APIs und dieser Doku.

## 08.11.2026

- Geändert: Fehlerantworten (`application/problem+json`) tragen jetzt einen stabilen `code`, einen `type` als URI auf den neuen [Fehlerkatalog](/api/errors/) (statt `about:blank`) und wo sinnvoll `param` und `allowed`. `title` und `detail` sind englisch, mit `Accept-Language: de` deutsch. Nicht lesbare Pfadkoordinaten (z. B. `/v1/parcel/abc`) ergeben `400` statt `404`.
- Geändert: Die OpenAPI-Beschreibung ist OpenAPI 3.1, vollständig englisch, mit Antwort-Schemas und Beispielen für die wichtigsten Endpunkte, Server-Liste, Rate-Limit-Beschreibung und `/v1/geocode/{q}`.
- Geändert: `overview` bei Directions ist standardmäßig `simplified` (wie Mapbox, deutlich kleinere Antworten bei langen Routen); `overview=full` liefert wie bisher alle Stützpunkte.
- Neu: Header `Server-Timing` (`router`, `search`, `render`, `total`), `ETag` und `Last-Modified` für Stile, TileJSON und OpenAPI (bedingte Anfragen mit `304`), `RateLimit-Policy` je Endpunktgruppe.
- Entfernt: die früheren Alias-Pfade `/v1/{tileset}.json` und `/v1/{tileset}/{z}/{x}/{y}.{ext}` (nutzen Sie `/v1/tiles/…`) sowie `/v1/directions/driving-ev/…`. Beide waren am Gateway bereits gesperrt (`404`).

- Deprecated: Die deutschen Bezeichner der API werden durch englische ersetzt. Die alten Namen funktionieren als Alias **bis zum 31.05.2027** (Sunset) und liefern die Header `Deprecation`, `Sunset` und `Link: …; rel="successor-version"`. Alle Zuordnungen: [Migration auf englische Namen](/api/migration/).
  - Stil-IDs: `karte` → `streets`, `karte-morgen` → `streets-dawn`, `karte-abend` → `streets-dusk`, `karte-nacht` → `streets-night`, `karte-verkehr` → `traffic-day`, `luftbild` → `satellite`, `hybrid` → `satellite-streets`, `gelaende` → `outdoors`.
  - Kachelsätze: `basis` → `streets`, `kataster` → `cadastre`, `hoehenlinien` → `contours`, `relief` → `terrain-rgb`, `dop` → `orthophoto`, `dop_flugblock` → `orthophoto-flight-blocks` (`dtm`, `dsm` bleiben).
  - Suchkategorien: 49 neue IDs in snake_case, zum Beispiel `bahnhof` → `train_station`, `apotheke` → `pharmacy`, `schule` → `school`.
  - Felder: Grundstück `gstnr`/`kgnr`/`bauflaeche`/`quelle` → `parcel_number`/`cadastral_municipality_id`/`is_building_land`/`source`, Höhe `stichtag` → `reference_date` (bis zum Sunset stehen beide Felder in der Antwort).
  - Stil-Metadaten: `mapx:stil` → `mapx:style`, `mapx:api-stil` → `mapx:api_style`.

## 07.11.2026

- Neu: Sentinel-2-Satellitenbild für Österreich (Best-Pixel-Composite, täglich fortgeschrieben, z5 bis z13) als Tileset `sentinel2`.
- Neu: ÖV mit aktuellen und Folgejahr-Fahrplänen, Haltestellen feedübergreifend zusammengeführt (IFOPT), Abfahrtsmonitor `/v1/departures` und Haltestellen `/v1/stops`.
- Neu: Maut in Directions (`exclude=toll`, `routes[].toll`, `toll_road`, `toll_road_m`).
- Neu: Tilesets `rastanlagen` (ASFINAG-Rastanlagen) und `sharing` (Shared-Mobility-Standorte), Suchkategorien `rastplatz` und `sharing`.
- Neu: Verkehrsmeldungen `/v1/traffic/*` (Ereignisse, Wechselverkehrszeichen) und `incidents[]` in Directions.
- Geändert: Reverse-Geocoding antwortet in 3 bis 60 ms statt rund 1 s; XYZ-Kacheln unter `/v1/tiles`; Attribution in den Antworten; `operationId` je Endpunkt in der OpenAPI.

- Neu: [Wetter](/api/weather/) (`/v1/weather/current`, `/v1/weather/forecast`, Wetterkacheln mit TileJSON und Zeitdimension) aus offenen Daten der GeoSphere Austria, CC BY 4.0; Beispiel [Wetter-Overlay](/beispiele/wetter-overlay/).

- Neu: [Static Images](/api/static/) (`/v1/static`, Luftbild und Gelände als PNG oder WebP mit Overlays), [SDK mapx-gl](/api/sdk/); englische Doku auf Stand der deutschen.

- Neu: Bereich [Beispiele](/beispiele/) mit zwölf lauffähigen Live-Beispielen; API-Seiten für [Styles](/api/styles/), [Geocoding](/api/geocoding/) und [Routing](/api/routing/) (alle Endpunkte unter `https://api.mapx.at/v1/` live).

- Neue Produktseiten: DSM, Höhenlinien, Basiskarte, Kataster, Orthophoto und Flugblöcke, jeweils mit Live-Karte und Layer-Tabelle.
- API-Endpunkte in Betrieb: Kacheln (PMTiles und XYZ), TileJSON, Höhe und Profil.
- Neuer Aufbau der Dokumentation, jede Seite auch als Markdown.

## Datenstände

<table>
  <thead><tr><th>Produkt</th><th>Stichtag</th><th>Datei</th><th>Erzeugt</th></tr></thead>
  <tbody>
    {Object.values(data.tilesets).filter((x) => ["relief", "dsm"].includes(x.key)).map((x) => (
      <tr><td>{x.tileset}</td><td>{x.stand}</td><td><code>{x.datei}</code></td><td>{x.erzeugt ?? ""}</td></tr>
    ))}
  </tbody>
</table>
# Changelog

Änderungen an den APIs, Tilesets und dieser Dokumentation, gegliedert in Neu, Geändert, Veraltet und Entfernt.

Deprecations tragen einen Stichtag: Ab diesem Tag wird der alte Name entfernt. Details zur Umbenennung: [Migration](/de/api/migration/).

## 2026-11-08

### Neu
- Header `Server-Timing`, `ETag`, `Last-Modified` (Styles, TileJSON, OpenAPI; bedingte Anfragen antworten mit `304`) und `RateLimit-Policy` je Endpunktgruppe.
- Lkw-Profil `driving-truck` mit `weight`, `height`, `width`, `length` und `truck_type`.
- Fehlerkatalog mit stabilem `code` je Fehler: [Fehler und Limits](/de/api/errors/).
- Doku: API-Referenz aus OpenAPI generiert, Playground je API, Guides je Produkt.

### Geändert
- Englische Bezeichner für Styles, Tilesets, Suchkategorien und Antwortfelder.
- Fehler sind `application/problem+json` mit `type`-URI, `code`, `param` und `allowed`; Text englisch, deutsch mit `Accept-Language: de`.
- Die OpenAPI-Beschreibung ist OpenAPI 3.1 mit Antwortschemas und Beispielen.
- Bei Directions ist `overview` standardmäßig `simplified`; `overview=full` liefert jeden Stützpunkt.

### Veraltet (Deprecated, Stichtag 2027-05-31)
- Style-IDs `karte`, `luftbild`, `gelaende`, `hybrid` und die übrigen deutschen IDs.
- Tileset-Namen `basis`, `kataster`, `hoehenlinien`, `relief`, `dop`, `dop_flugblock`.
- Die 49 deutschen Suchkategorie-IDs (zum Beispiel `bahnhof`, `apotheke`).
- Grundstücksfelder `gstnr`, `kgnr`, `bauflaeche`, `quelle` und Höhenfeld `stichtag` (bis zum Stichtag stehen beide Namen in der Antwort).

### Entfernt
- Die Alias-Pfade `/v1/{tileset}.json` und `/v1/{tileset}/{z}/{x}/{y}.{ext}`. Stattdessen `/v1/tiles/…` verwenden.

## 2026-11-07

### Neu
- Öffentlicher Verkehr: Verbindungen (`/v1/directions/transit`), Abfahrten (`/v1/departures`) und Haltestellen (`/v1/stops`).
- Maut in Directions: `exclude=toll`, `routes[].toll`, `toll_road`, `toll_road_m`.
- Verkehr: `/v1/traffic/*` (Störungen, Tempolimits, Zeichen, Tiles), `incidents` in Directions, Profil `driving-traffic`.
- Tilesets `rastanlagen` (Rastanlagen) und `sharing` (Sharing-Mobilität), Suchkategorien für beide.
- Wetter: Beobachtungen, stündliche Vorhersage und Wetter-Tiles (GeoSphere Austria, CC BY 4.0).
- Static Images: Luftbilder, Gelände- und Kartenstyles als PNG oder WebP mit Overlays.
- Tilequery, Parcel, Search Box, Address Autofill und Address Validation, Batch-Geocoding.
- Directions: `alternatives`, `annotations`, `radiuses`; `matrix`, `isochrone`, `matching`, optimierte Touren und Tourenplanung.
- SDK mapx-gl.

### Geändert
- Reverse Geocoding antwortet in Millisekunden statt in etwa einer Sekunde.
- XYZ-Tiles wurden unter `/v1/tiles` verschoben.
- Attribution in den Antworten; `operationId` an jedem Endpunkt in der OpenAPI.

## Früher

### Neu
- Tilesets DTM, DSM, Höhenlinien, Basiskarte, Kataster, Orthophoto und Flugblöcke als PMTiles, TileJSON und XYZ.
- Höhen- und Profil-API.
- Dokumentation auf Deutsch und Englisch.

## Tileset-Stände

<table>
  <thead><tr><th>Tileset</th><th>Stand</th><th>Datei</th></tr></thead>
  <tbody>
    {Object.values(data.tilesets).filter((x) => ["relief", "dsm"].includes(x.key)).map((x) => (
      <tr><td>{x.key === "relief" ? "terrain-rgb" : x.key}</td><td>{x.stand}</td><td><code>{x.datei}</code></td></tr>
    ))}
  </tbody>
</table>
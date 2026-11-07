# Changelog

Änderungen an Datenprodukten, APIs und dieser Doku.

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
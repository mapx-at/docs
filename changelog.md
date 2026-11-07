# Changelog

Änderungen an Datenprodukten, APIs und dieser Doku.

## 07.11.2026

- Neu: Bereich [Beispiele](/beispiele/) mit zwölf lauffähigen Live-Beispielen; API-Seiten für [Styles](/api/styles/), [Geocoding](/api/geocoding/) und [Routing](/api/routing/) (Endpunkte unter `/v1/` werden ausgerollt).

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
# Changelog

Änderungen an Datenprodukten, APIs und dieser Doku.

## 07.11.2026

- Doku-Website gestartet (Deutsch, DTM-Seite auch auf Englisch).
- Spezifikation der Endpunkte `v1/{tileset}.json`, `v1/{tileset}/{z}/{x}/{y}.webp` und `elevation` veröffentlicht. Verfügbarkeit: siehe hier, sobald die Endpunkte in Betrieb sind.
- `relief/README.md` korrigiert: Quell-Nodata wird nicht als Höhe 0 kodiert, sondern per Füllraster glatt fortgesetzt.

## Datenstände

<table>
  <thead><tr><th>Produkt</th><th>Stichtag</th><th>Datei</th><th>Erzeugt</th></tr></thead>
  <tbody>
    {Object.values(data.tilesets).filter((x) => ["relief", "dsm"].includes(x.key)).map((x) => (
      <tr><td>{x.tileset}</td><td>{x.stand}</td><td><code>{x.datei}</code></td><td>{x.erzeugt ?? ""}</td></tr>
    ))}
  </tbody>
</table>
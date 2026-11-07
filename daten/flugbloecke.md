# Flugblöcke

Welcher Bildflug liefert welches Orthophoto-Pixel: Gewinnerflächen der BEV-Operate mit Flugzeitraum, als Vektorkacheln z6 bis z17.

export const t = data.tilesets.dop_flugblock;
export const url = `https://tiles.mapx.at/${t.datei}`;

Zu jedem Pixel des [Orthophotos](/daten/orthophoto/) die Fläche des Bildflugs (Operats), aus dem es stammt, mit Flugzeitraum.
Damit lässt sich zum Beispiel „Aufnahme von“ in einer Karte anzeigen.

## Kennwerte

<Kennwerte id="dop_flugblock" />

## Live-Karte

<ProduktKarte id="dop_flugblock" center={[14.3, 47.6]} zoom={7} />

## Layer

<LayerTabelle id="dop_flugblock" info={{ dop_flugblock: "Gewinnerflächen je Operat" }} />

| Feld | Bedeutung |
|---|---|
| `operat` | Name des Operats (z. B. Zillertal, Linz) |
| `flugblock_id`, `archivnr` | Archivnummer des Operats (Text bzw. Zahl) |
| `flug_von`, `flug_bis` | Beginn und Ende des Befliegungszeitraums |
| `flugdatum` | Ende des Zeitraums, bestimmt den Vorrang |
| `flugjahr` | Jahr aus dem Datum |
| `rang` | Vorrang, 0 = höchster |
| `stichtag` | Lieferstand der Bilddaten, nicht das Flugdatum |

Ein Flugblock ist ein **Operat**, kein einzelner Flugtag: Es gibt kein Tagesdatum, nur den Zeitraum des Operats; ein Operat kann bis zu drei Monate dauern.

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/dop-flugblock.json" } },
    layers: [{ id: "dop_flugblock", type: "line", source: "v", "source-layer": "dop_flugblock", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/dop-flugblock/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/dop-flugblock.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/dop-flugblock/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Datenherkunft

- Quelle: BEV „Aktualität Orthophoto Operate Farbe und Infrarot“.
- Die Flächen zeigen, wo ein Operat im Orthophoto **tatsächlich** zu sehen ist, nicht die ursprünglichen Umrisse. Ein Operat kann mehrere Flächen haben.

## Hinweise

- Der Vorrang folgt dem Ende des Zeitraums und kann überraschen: St. Pölten (Ende 23.07.2023) liegt vor Linz (28.05.2023).

## Versionen

Datei mit Stichtag im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dop_flugblock.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
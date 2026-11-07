# Flugblöcke

Welcher Bildflug liefert welches Orthophoto-Pixel: Gewinnerflächen der BEV-Operate mit Flugzeitraum, als Vektorkacheln z6 bis z17.

export const t = data.tilesets.dop_flugblock;
export const url = `https://tiles.mapx.at/${t.datei}`;

Zu jedem Pixel des [Orthophotos](/daten/orthophoto/) die Fläche des Bildflugs (Operats), aus dem es stammt, mit Flugzeitraum und Quell-Mosaik.
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
| `mosaik_url` | URL des Quell-Mosaiks (COG) beim BEV |
| `stichtag` | Lieferstand des Mosaiks (aus der URL), nicht das Flugdatum |
| `fid` | laufende Nummer |

Ein Flugblock ist ein **Operat**, kein einzelner Flugtag: Das BEV liefert nur Operat-Umrisse, ein Operat kann bis zu drei Monate dauern.
Die Beschreibung im Tileset („Operat × Flugtag“) ist in diesem Punkt ungenau.

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

- Metadaten: BEV „Aktualität Orthophoto Operate Farbe und Infrarot“, 32 Operate.
- Die Flächen kommen aus dem Besitzerraster des Orthophoto-Renderers, nicht aus den BEV-Umrissen: Sie zeigen, wo das Operat **tatsächlich** gewonnen hat. Ein Operat kann mehrere Flächen haben.
- Eckpunkte liegen auf z18-Pixelecken; bis z15 pixelgenau, z16 und z17 fein unterteilt.

## Genauigkeit und Grenzen

- Geprüft: 0 Abweichungen Pixel gegen Polygon (35 400 Pixel, 830 000 Randpixel); die Fläche je Operat wird beim Lauf gegen die Pixelzahl geprüft.
- Kein Tagesdatum, nur der Zeitraum des Operats.
- Der Vorrang folgt dem Ende des Zeitraums und kann überraschen: St. Pölten (Ende 23.07.2023) liegt vor Linz (28.05.2023).

## Versionen

Datei mit Stichtag und Hash im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei | SHA-256 |
|---|---|---|
| {t.stand} | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dop_flugblock.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
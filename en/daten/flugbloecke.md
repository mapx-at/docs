# Flight blocks

Which flight supplies which orthophoto pixel: winning areas of the BEV surveys with flight period, as vector tiles z6 to z17.

export const t = data.tilesets.dop_flugblock;
export const url = `https://tiles.mapx.at/${t.datei}`;

For every pixel of the [Orthophoto](/en/daten/orthophoto/), the area of the flight (survey) it comes from, with flight period and source mosaic.
This lets you show, for example, "captured on" in a map. Use cases: capture date of imagery, quality and currency checks, change detection between surveys.

## Key figures

<Kennwerte id="dop_flugblock" lang="en" />

## Live map

<ProduktKarte id="dop_flugblock" center={[14.3, 47.6]} zoom={7} lang="en" />

## Layers

<LayerTabelle id="dop_flugblock" lang="en" info={{ dop_flugblock: "Winning areas per survey" }} />

| Field | Meaning |
|---|---|
| `operat` | Name of the survey (e.g. Zillertal, Linz) |
| `flugblock_id`, `archivnr` | Archive number of the survey (text and number respectively) |
| `flug_von`, `flug_bis` | Start and end of the flight period |
| `flugdatum` | End of the period, determines precedence |
| `flugjahr` | Year from the date |
| `rang` | Precedence, 0 = highest |
| `fid` | Sequential number |

A flight block is a **survey** (Operat), not a single flight day: a survey can take up to three months.

## Usage

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
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/en/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/dop-flugblock/{z}/{x}/{y}.pbf`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/dop-flugblock.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/dop-flugblock/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Data origin

- Source: BEV "Currency of orthophoto surveys, color and infrared" (Aktualität Orthophoto Operate Farbe und Infrarot).
- The areas show where a survey is actually visible in the orthophoto. A survey can have several areas.

## Notes

- Date information is the period of the survey, not a daily date.
- Precedence follows the end of the period and can differ from the order of the survey start.

## Versions

The file name carries the reference date and hash and never changes; a new version is a new file.

| Version | File |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

All changes: [Changelog](/en/changelog/).

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.dop_flugblock.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/en/lizenz/).
# Contour lines

Contour lines for Austria as vector tiles, z10 to z16, 100 m to 10 m interval, derived from the BEV ALS-DTM 1 m.

export const t = data.tilesets.hoehenlinien;
export const url = `https://tiles.mapx.at/${t.datei}`;

Contour lines from the [DTM](/en/daten/dtm/), in four generalization levels. Each level has its own interval, its own smoothing and its own index lines.

Use cases: hiking and outdoor maps, topographic maps, planning and terrain analysis.

## Key figures

<Kennwerte id="hoehenlinien" lang="en" />

## Live map

<ProduktKarte id="hoehenlinien" center={[12.694, 47.074]} zoom={13} lang="en" />

## Layers

<LayerTabelle id="hoehenlinien" lang="en" info={{ contour_l: "<code>ele</code> elevation in m, <code>index</code> 1 = index line, <code>lvl</code> level (10, 12, 14, 16)" }} />

## Levels

| `lvl` | Zoom | Interval | Index line | Grid | Smoothing (Gaussian σ) | Minimum length |
|---|---|---|---|---|---|---|
| 10 | 10–11 | 100 m | 500 m | 20 m | 60 m | 600 m |
| 12 | 12–13 | 50 m | 200 m | 10 m | 25 m | 250 m |
| 14 | 14–15 | 20 m | 100 m | 4 m | 10 m | 100 m |
| 16 | from 16 | 10 m | 50 m | 2 m | 4 m | 40 m |

There are no lines below z10. Filter by `lvl` in the style, otherwise several levels overlap at z12 and above
if the client overzooms.

<Code lang="js" code={`{ id: "hl", type: "line", source: "v", "source-layer": "contour_l",
  paint: { "line-color": "#b5651d", "line-width": ["case", ["==", ["get", "index"], 1], 1.4, 0.6] } }
{ id: "hl-text", type: "symbol", source: "v", "source-layer": "contour_l",
  filter: ["==", ["get", "index"], 1],
  layout: { "symbol-placement": "line", "text-field": ["concat", ["get", "ele"], " m"], "text-size": 10 } }`} />

## Usage

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/hoehenlinien.json" } },
    layers: [{ id: "contour_l", type: "line", source: "v", "source-layer": "contour_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/en/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/hoehenlinien/{z}/{x}/{y}.pbf`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/hoehenlinien.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/hoehenlinien/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Data origin

- Source: BEV ALS-DTM 1 m, reference date 15 Sep 2025 (same as the DTM).
- Per level the terrain is generalized and smoothed, so the lines stay clean and legible at every scale.
- Standing waters of 1 ha or more are masked; the lines end at the shore.

## Accuracy and notes

- Because of the smoothing, the line at coarse levels deliberately does not lie exactly on the raw elevation.
- Lines end at the area edge (national border).
- On steep faces 10 m lines are very dense; thinning is the style's job.
- Labeling via the attribute `ele`; there are no dedicated label points.

## Versions

The file name carries the reference date and hash and never changes; a new version is a new file.

| Version | File |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

All changes: [Changelog](/en/changelog/).

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.hoehenlinien.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/en/lizenz/).
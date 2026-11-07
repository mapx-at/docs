# Cadastre

Parcel boundaries from the BEV Digital Cadastral Map as vector tiles, z16, with cadastral municipality and parcel number.

export const t = data.tilesets.kataster;
export const url = `https://tiles.mapx.at/${t.datei}`;

The boundaries of all parcels as lines, with cadastral municipality number and parcel number. The tileset has only z16; clients scale up beyond that.

Use cases: property search, planning and building applications, overlay on orthophoto or base map.

<Aside type="caution">The display is not legally binding. For boundary questions the cadastre at the survey office applies.</Aside>

## Key figures

<Kennwerte id="kataster" lang="en" />

## Live map

<ProduktKarte id="kataster" center={[15.334, 48.227]} zoom={16.5} lang="en" />

## Layers

<LayerTabelle id="kataster" lang="en" info={{ parcel_l: "<code>kgnr</code> cadastral municipality number, <code>gstnr</code> parcel number (building parcels with a dot, e.g. <code>.211</code>)" }} />

The lines are boundaries, not areas: a click hits the boundary line and returns the parcel it belongs to. Owner and area are not included.

## Usage

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/mapx-kataster.json" } },
    layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/en/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/mapx-kataster/{z}/{x}/{y}.pbf`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/mapx-kataster.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/mapx-kataster/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

Because only z16 exists, set `minzoom: 16` in the style; the source with `maxzoom: 16` overzooms automatically.

## Data origin

- Source: BEV **Digital Cadastral Map (DKM)**, reference date 1 Apr 2026.
- Position via the official transformation grid (NTv2) in ETRS89.

## Notes

- The display is not legally binding; the depiction follows the digital cadastral map.
- There are no tiles below z16.

## Versions

The file name carries the reference date and hash and never changes; a new version is a new file.

| Version | File |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

All changes: [Changelog](/en/changelog/).

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.kataster.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/en/lizenz/).
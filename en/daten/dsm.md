# DSM surface model

Digital surface model of Austria including buildings and vegetation as Terrain-RGB (0.1 m), z5 to z16, from the BEV ALS-DOM 1 m.

export const t = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

The DSM is the counterpart to the [DTM](/en/daten/dtm/): it shows the **surface** with buildings, trees and bridges. Encoding, tile size and zoom levels are identical,
so all DTM clients and formulas apply unchanged.

Use cases: building and vegetation heights (together with the DTM), line-of-sight and visibility analysis, solar and shading planning, 3D city and landscape models.

## Key figures

<Kennwerte id="dsm" lang="en" />

## Live map

<DtmMap id="dsm" lang="en" />

Clicking decodes the tile pixel (formula under [Terrain-RGB](/en/konzepte/terrain-rgb/)).

## Usage

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/dsm.json", tileSize: 512, encoding: "mapbox" } },
    terrain: { source: "dsm", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/en/start/)).
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/dsm/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="QGIS">
    XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/dsm/{z}/{x}/{y}.webp`. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/dsm.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/dsm/14/8860/5730.webp`} />
  </TabItem>
</Tabs>

Elevation over HTTP: [Elevation and profile](/en/api/hoehe/) with `model=dsm`.

## Data origin

- Source: BEV **ALS-DOM 1 m** (airborne laser scanning, surface), reference date **15 Sep 2025**.
- Encoding 0.1 m (Mapbox), 512 px, lossless WebP, in Web Mercator.
- Overview levels z15 to z5 are averaged in elevation space.

## Accuracy and notes

- Quantization 0.1 m. The finest level is z16 (about 0.8 m per pixel at 48° N); beyond that the client scales up.
- Accuracy corresponds to the official BEV source.
- Buildings and vegetation reflect the time of the survey flight, not the reference date.
- Outside the data the value is a smooth continuation, not a measurement (as with the DTM).
- Building and vegetation height (nDSM = DSM − DTM) can be computed in the client from both tile sets.

## Versions

The file name carries the reference date and hash and never changes; a new version is a new file.

| Version | File |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

All changes: [Changelog](/en/changelog/).

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.dsm.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/en/lizenz/).
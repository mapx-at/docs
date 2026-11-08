# DSM surface model

Digital surface model of Austria including buildings and vegetation as Terrain-RGB (0.1 m), z5 to z16, from the BEV ALS-DOM 1 m.

export const t = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

The DSM is the counterpart to the [DTM](/tilesets/terrain-rgb/): it shows the surface including buildings, trees and bridges. Encoding, tile size and zoom levels are identical; all clients and formulas of the DTM apply unchanged.

<Nutzungen items={[{t:"Building and vegetation height",d:"nDSM as DSM minus DTM in the client"},{t:"Solar planning",d:"Roof and shading heights as a basis"},{t:"Visibility and shading",d:"Surface including buildings and forest"},{t:"3D city and landscape views",d:"Surface as terrain in the browser",href:"/guides/3d-terrain/"},{t:"Elevation profiles",d:"Profile over the surface",href:"/guides/elevation-profile/"}]} />

## Specifications

<Kennwerte id="dsm" lang="en" />

## Live map

<DtmMap id="dsm" lang="en" />

Clicking decodes the tile pixel (formula under [Terrain-RGB](/reference/terrain-rgb/)).

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/dsm.json", tileSize: 512, encoding: "mapbox" } },
    terrain: { source: "dsm", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/start/)).
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="QGIS">
    XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp`. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/tiles/dsm.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/tiles/dsm/14/8860/5730.webp`} />
  </TabItem>
</Tabs>

Elevation over HTTP: [Elevation and profile](/api/elevation/) with `model=dsm`.

## Source and currency

The DSM is derived from the BEV ALS-DOM 1 m (airborne laser scanning, surface). The reference date is in the specifications and the file name; a file never changes, a new release is a new file.

- Source: BEV ALS-DOM 1 m, reference date 15 Sep 2025.
- Same encoding as the DTM: Mapbox Terrain-RGB with 0.1 m quantization, 512 px, lossless WebP.

## Notes

- Quantization 0.1 m. The finest level is z16 (about 0.8 m per pixel at 48° N); beyond that the client scales up.
- Accuracy corresponds to the official BEV source.
- Buildings and vegetation reflect the time of the survey flight, not the reference date.
- Outside the data the value is a smooth continuation, not a measurement (as with the DTM).
- Building and vegetation height (nDSM = DSM − DTM) can be computed in the client from both tile sets.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.dsm.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/reference/attribution/).
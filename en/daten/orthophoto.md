# Orthophoto

Aerial imagery of Austria from the BEV as WebP tiles, z5 to z18, 20 cm, with an unambiguous flight-block choice per pixel.

export const t = data.tilesets.dop;
export const url = `https://tiles.mapx.at/${t.datei}`;

The BEV orthophoto, re-tiled in Web Mercator up to z18 (20 cm, the native resolution of the source). Which flight supplies a pixel is shown by the [Flight blocks](/en/daten/flugbloecke/) tileset.

<Nutzungen items={[{t:"Aerial imagery background",d:"Satellite-map alternative for web and apps"},{t:"3D terrain",d:"Orthophoto on the DTM",href:"/en/anleitungen/gelaende-3d/"},{t:"Planning and documentation",d:"Situation on site at 20 cm"},{t:"Checking parcels",d:"Cadastre over the aerial image"},{t:"QGIS",d:"Aerial imagery as XYZ or PMTiles layer",href:"/en/anleitungen/qgis/"}]} />

## Specifications

<Kennwerte id="dop" lang="en" />

## Live map

<ProduktKarte id="dop" overlay="dop_flugblock" center={[15.334, 48.227]} zoom={14} lang="en" />

Clicking shows the flight block at the point (survey, flight period).

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { ortho: { type: "raster", url: "https://tiles.mapx.at/v1/orthophoto.json", tileSize: 512 } },
    layers: [{ id: "ortho", type: "raster", source: "ortho" }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/en/start/)).
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/orthophoto/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="QGIS">
    XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/orthophoto/{z}/{x}/{y}.webp`. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/orthophoto.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/orthophoto/14/8860/5730.webp`} />
  </TabItem>
</Tabs>

## Source and currency

- Source: BEV **Digital Orthophoto (DOP) RGB**, 20 cm, captured 2022 to 2025.
- Georeferencing via the official transformation grid (ETRS89 ↔ MGI).
- Overview levels z17 to z5 are averaged from z18. Empty tiles are omitted.
- Where surveys overlap, the more recent one is shown (no blending).

## Notes

- All of Austria is covered.
- Color seams between surveys are visible; there is no color matching.
- At the borders to DE, CH, LI, IT, SI, CZ and SK the imagery partly extends abroad; at the border to Hungary it ends at the border.
- Below z18, pixels of both surveys mix at block boundaries; the assignment is unambiguous at z18.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.dop.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/en/lizenz/).
# Sentinel-2 current

Always up-to-date, cloud-free satellite image of Austria from Copernicus Sentinel-2 as WebP tiles, z5 to z13, 10 m, with the acquisition date per pixel.

A satellite image of Austria that updates every day: each pixel shows a **robust value from the most recent cloud- and shadow-free Sentinel-2 acquisitions** (rolling best-pixel composite). 10 m resolution, Web Mercator, WebP, 512 px, z5 to z13 (clients overzoom beyond). The acquisition date of every pixel is kept in the store.

<Nutzungen items={[{t:"Current backdrop",d:"A satellite view that is not years old"},{t:"See change",d:"Snow cover, harvest, windthrow, building sites at 10 m"},{t:"Overview and planning",d:"Landscape, vegetation and water at a glance"}]} />

## Embedding

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { s2: { type: "raster", url: "https://api.mapx.at/v1/tiles/sentinel2.json", tileSize: 512 } },
    layers: [{ id: "s2", type: "raster", source: "s2" }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution. The ready-made style `satellit-aktuell` (Sentinel-2 with roads and names) is served at `/v1/styles/satellit-aktuell`.
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://api.mapx.at/v1/tiles/sentinel2.json
curl -s -o tile.webp https://api.mapx.at/v1/tiles/sentinel2/13/4444/2839.webp`} />
  </TabItem>
</Tabs>

> The tileset is replaced daily (fixed tileset `sentinel2`, no versioned file name). TileJSON reports the state in `mapx:stand` (most recent scene included); PMTiles directly: `https://tiles.mapx.at/sentinel2.pmtiles` (5 minute cache).

## Source and method

- Source: **Sentinel-2 L2A** (bottom-of-atmosphere reflectance) by ESA/Copernicus as Cloud-Optimized GeoTIFF via Element84 Earth Search (STAC, AWS Open Data). Only bands B04, B03, B02 (10 m) and the scene classification SCL (20 m) are read via HTTP range requests.
- **Mask:** valid SCL classes are 4 (vegetation), 5 (not vegetated), 6 (water) and 11 (snow); clouds, shadows, cirrus, dark and unclassified pixels do not count. The mask is eroded by 40 m, and pixels with a very high blue share (haze) are dropped, except snow.
- **Robust value instead of the latest value:** per pixel the store keeps the last 5 valid observations. The output takes, among the observations of the 45 days before the most recent one, the value of median brightness; outliers (too blue, too bright, too dark) against the median are discarded. The image stays current while haze and thin cloud drop out.
- **Gaps:** older observations (up to 180 days) fill pixels that were never clear recently; remaining gaps are filled from neighbouring pixels.
- **Seams:** each scene is matched in brightness to the store, and edges between acquisition days are blended softly.
- The **acquisition date per pixel** is kept in the store; the metadata field `newest` names the most recent scene included.

## Notes

- The image is a mix of different days (date per pixel in the store); dense ground fog lasting weeks may remain despite the filter.
- 10 m source resolution: z13 is the last level with real detail gain.

## Attribution

Required by the Copernicus terms (free, full and open) for modified data:

<Code lang="html" code={"Contains modified Copernicus Sentinel data 2026"} />

The year is the year of the acquisitions; MapLibre takes the text from TileJSON or PMTiles. More: [Licence](/en/lizenz/).
# Sentinel-2 current

Always up-to-date, cloud-free satellite image of Austria from Copernicus Sentinel-2 as WebP tiles, z5 to z13, 10 m, updated daily at 02:00.

A satellite image of Austria that updates every day: each pixel shows a **robust value from the most recent cloud- and shadow-free Sentinel-2 acquisitions** (rolling best-pixel composite). Resolution 10 m, Web Mercator, WebP, 512 px, z5 to z13 (the client overzooms beyond that). The acquisition date of every pixel is kept in the underlying store.

<Nutzungen items={[{t:"Current background",d:"Satellite view that is not years old"},{t:"Seeing change",d:"Snow cover, harvest, windthrow, construction sites on a 10 m grid"},{t:"Overview and planning",d:"Landscape, vegetation and water at a glance"}]} />

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  center: [15.334, 48.227], zoom: 11,
  style: {
    version: 8,
    sources: { s2: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/sentinel2.json", tileSize: 512 } },
    layers: [{ id: "s2", type: "raster", source: "s2" }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. The ready-made style `satellit-aktuell` (Sentinel-2 with roads and labels) is available at `/v1/styles/satellit-aktuell`.
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/tiles/sentinel2/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN", {
  tileSize: 512, zoomOffset: -1, minZoom: 5, maxNativeZoom: 13, maxZoom: 18, crossOrigin: true,
  attribution: "Contains modified Copernicus Sentinel data 2026"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/sentinel2.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o tile.webp https://tiles.mapx.at/v1/tiles/sentinel2/13/4444/2839.webp`} />
  </TabItem>
</Tabs>

> The tileset is replaced daily (fixed tileset `sentinel2`, no versioned file name). The TileJSON states the date in the field `mapx:stand` (most recent ingested scene). PMTiles directly: `https://tiles.mapx.at/sentinel2.pmtiles?access_token=YOUR_TOKEN` (cached for 5 minutes).

## TileJSON, zoom and update

| | |
|---|---|
| TileJSON | `https://tiles.mapx.at/v1/tiles/sentinel2.json` |
| Tiles | `https://tiles.mapx.at/v1/tiles/sentinel2/{z}/{x}/{y}.webp` |
| Zoom | z5 to z13 (6.4 m per pixel at z13 on 47.5 degrees north; the source is 10 m) |
| Format | WebP, 512 px, Web Mercator |
| Coverage | Austria (tiles at the border extend abroad) |
| Update | daily at 02:00; new scenes are merged into the store, then the file is replaced atomically |

## Source and method

- Source: **Sentinel-2 L2A** (surface reflectance) from ESA/Copernicus as Cloud-Optimized GeoTIFF via Element84 Earth Search (STAC, AWS Open Data). Only bands B04, B03, B02 (10 m) and the scene classification SCL (20 m) are read, with HTTP range requests.
- **Mask:** SCL classes 4 (vegetation), 5 (bare soil), 6 (water) and 11 (snow) are valid; clouds, shadows, cirrus, dark and unclassified pixels do not count. The mask is eroded by 40 m, and pixels with a very high blue share (haze) are dropped, except snow.
- **Robust value instead of latest value:** each pixel keeps its last 5 valid observations. The output is a value of medium brightness among the observations of the 45 days before the most recent one; outliers (too blue, too bright, too dark) against the median are discarded. The image stays current while haze and thin clouds drop out.
- **Gaps:** older observations fill pixels that were never clear recently; remaining holes are filled from neighbouring pixels.
- **Seams:** each scene is brightness-matched to the stored data, and edges between acquisition days are blended softly.

## Index and date overlays

Four further fixed tilesets are derived from the same composite. They are transparent where there is no value, so they work as overlays on `sentinel2`, a satellite image or a street map. All are WebP, 512 px, z5 to z13, same attribution; each is replaced daily together with `sentinel2`. They appear in `/v1/tiles` only once the file exists. The TileJSON carries the value range and colour ramp in `mapx:legend`.

| Tileset | Shows | Value and colour |
|---|---|---|
| `sentinel2-datum` | Acquisition date per pixel | age in days before `mapx:stand`: 0 green, 20 yellow, 45 orange, 90 red, 180 dark red |
| `sentinel2-ndvi` | Vegetation, (NIR - red) / (NIR + red) | -0.1 grey, 0.05 bare, 0.2 sand, 0.4 light green, 0.85 dark green |
| `sentinel2-ndwi` | Water, (green - NIR) / (green + NIR) | only values above 0: light blue (0) to dark blue (0.7) |
| `sentinel2-ndsi` | Snow, (green - SWIR) / (green + SWIR) | only values from 0.4: light blue to white (0.8); water is excluded by a NIR threshold |

<Code lang="js" code={`map.addSource("ndvi", { type: "raster", url: "https://tiles.mapx.at/v1/tiles/sentinel2-ndvi.json", tileSize: 512 });
map.addLayer({ id: "ndvi", type: "raster", source: "ndvi", paint: { "raster-opacity": 0.8 } });`} />

In the mapx viewer they are the overlays "Acquisition date", "Vegetation (NDVI)", "Water (NDWI)" and "Snow (NDSI)" in the layer group Satellite, with legend and opacity slider.

## Notes

- The image mixes several days (the date is stored per pixel); dense ground fog lasting weeks can remain despite the filter.
- 10 m source resolution: z13 is the last level with real gain in detail.

## Attribution

Required under the Copernicus terms of use (free, full and open) for modified data:

<Code lang="html" code={"Contains modified Copernicus Sentinel data 2026"} />

The year is the year of acquisition; MapLibre picks up the text from TileJSON or PMTiles. More: [License](/reference/attribution/).
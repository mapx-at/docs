# DTM terrain model

Digital terrain model of Austria as Terrain-RGB tiles (Mapbox encoding, 0.1 m), z5 to z16, from the BEV ALS-DTM 1 m.

export const t = data.tilesets.relief;
export const dsm = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

The DTM is an elevation tile set for all of Austria: bare-earth height without buildings or vegetation, encoded as colour (Terrain-RGB),
usable as `raster-dem` for hillshade and 3D terrain in MapLibre, Mapbox GL and compatible clients.

## 1. Key facts

<Kennwerte id="relief" lang="en" />

Generated at build time from `viewer/tiles-manifest.json` and the PMTiles header of the file (generated {data.erzeugt}).

## 2. Live map

<DtmMap id="relief" lang="en" />

A click reads the tile of the current zoom level (max z16) with a range request, decodes the pixel at the click point with the formula in section 4 and shows elevation and RGB.

## 3. Using it

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`import * as maplibregl from "maplibre-gl";

maplibregl.addProtocol("pmtiles", new Protocol().tile);
const map = new maplibregl.Map({
  container: "map",
  center: [12.694, 47.074], zoom: 11, pitch: 60,
  style: {
    version: 8,
    sources: { dem: { type: "raster-dem", url: "pmtiles://${url}", tileSize: 512, encoding: "mapbox", maxzoom: 16 } },
    terrain: { source: "dem", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dem" }]
  }
});`} />
  </TabItem>
  <TabItem label="Python">
    <Code lang="python" code={`# pip install pmtiles pillow numpy requests
from PIL import Image
from pmtiles.reader import Reader

URL = "${url}"

def get_bytes(offset, length):
    r = requests.get(URL, headers={"Range": f"bytes={offset}-{offset + length - 1}"}, timeout=30)
    r.raise_for_status()
    return r.content

tile = Reader(get_bytes).get(12, 2192, 1439)
rgb = np.asarray(Image.open(io.BytesIO(tile)).convert("RGB")).astype(np.uint32)
elevation = -10000 + (rgb[..., 0] * 65536 + rgb[..., 1] * 256 + rgb[..., 2]) * 0.1`} />
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s -r 0-126 ${url} | xxd | head -4
curl -s -o tile.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp`} />
  </TabItem>
</Tabs>

Leaflet, OpenLayers and QGIS examples: see the [German page](/daten/dtm/#3-einbinden).

## 4. Encoding

```
elevation_m = -10000 + (R * 65536 + G * 256 + B) * 0.1
```

Values are rounded to 0.1 m and clamped to the range. In Austria they span about 115 m to about 3798 m. Lossless WebP, 512 x 512 px, no border (no 1 px skirt): use `tileSize: 512`.

## 5. Resolution per zoom

At 47.5 N, 512 px tiles:

<Aufloesung lang="en" />

z16 (about 0.8 m per pixel) is the first level that does not coarsen the 1 m source. From z17 the client overzooms the z16 tile (`maxzoom: 16`); no new information appears. z15 to z5 are 2x2 averages in height space.

## 6. Provenance

BEV ALS-DTM 1 m (Geonetwork series `5ce253fc-b7c5-4362-97af-6556c18a45d9`), 55 source tiles of 50 x 50 km as COG in EPSG:3035, reference date 15 Sep 2025, reprojected to EPSG:3857 bilinearly.

## 7. Accuracy and limits

Measured: quantisation 0.1 m; z16 versus the bilinearly interpolated source at three check points within +-0.1 m (-0.03, -0.04, +0.02 m); Rust render versus source on ten tiles at most 0.05 m (rounding).

Limits: bare earth only (no buildings, trees). **No measurements outside the data**: the tile set covers the rectangle 9.53-17.17 E, 46.37-49.02 N; pixels without source data are a smooth continuation of the terrain (mean error 11 m in a test outside the data near Melk). Deviation of overview levels z15-z5 from the source and country-wide RMSE: **not measured**.

## 8. Variants

DSM (surface incl. buildings and vegetation, same encoding): <code>{dsm.datei}</code>, {(dsm.bytes / 1e9).toFixed(1)} GB.

## 9. Versions

| Reference date | File | SHA-256 |
|---|---|---|
| 2025-09-15 | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Files never change; a new reference date is a new file.

## 10. Attribution

```
Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0
```

## 11. FAQ

**Why lossless WebP?** Any colour change is a different elevation.

**Why no nodata?** Missing pixels are filled with a smooth continuation (area mean, push-pull) before encoding, so hillshade and 3D show no holes. Tiles without any source data do not exist.

**How does it differ from Mapbox/MapTiler?** Same encoding as `mapbox-terrain-dem-v1` but without the 1 px border; Mapbox serves up to z14, mapx up to z16 (Austria only), key-free and downloadable as a whole file.
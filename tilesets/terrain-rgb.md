# DTM terrain model

Digital terrain model of Austria as Terrain-RGB tiles (Mapbox encoding, 0.1 m), z5 to z16, from the BEV ALS-DTM 1 m.

export const t = data.tilesets.relief;
export const dsm = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

The DTM is an elevation tile set for all of Austria: ground height without buildings and vegetation, encoded as color (Terrain-RGB). It works directly as `raster-dem` for hillshading and 3D terrain in MapLibre, Mapbox GL and compatible clients.

<Nutzungen items={[{t:"Hiking and outdoor maps",d:"Hillshade as a map background up to z16"},{t:"3D terrain",d:"Terrain and flight views with orthophoto",href:"/guides/3d-terrain/"},{t:"Elevation profiles",d:"Ascent and descent along a route",href:"/guides/elevation-profile/"},{t:"Slope and aspect analysis",d:"Gradient and orientation from the elevation values"},{t:"QGIS and analysis",d:"Elevation as raster in desktop GIS",href:"/guides/qgis/"}]} />

## Specifications

<Kennwerte id="relief" lang="en" />

## Live map

<DtmMap id="relief" lang="en" />

Clicking reads the tile of the current zoom level (z16 at most) via range request, decodes the pixel at the click point with the
formula under “Encoding” and shows elevation and RGB value. With 3D enabled, the value of `queryTerrainElevation` is shown as well.

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`import * as maplibregl from "maplibre-gl";

maplibregl.addProtocol("pmtiles", new Protocol().tile);
const map = new maplibregl.Map({
  container: "map",
  center: [12.694, 47.074], zoom: 11, pitch: 60,
  style: {
    version: 8,
    sources: { dem: { type: "raster-dem", url: "pmtiles://${url}?access_token=YOUR_TOKEN", tileSize: 512, encoding: "mapbox", maxzoom: 16 } },
    terrain: { source: "dem", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dem" }]
  }
});
map.on("click", (e) => console.log(map.queryTerrainElevation(e.lngLat)));`} />
  </TabItem>
  <TabItem label="QGIS">
    1. In the Browser panel, right-click **XYZ Tiles**, **New Connection**.
    2. URL `https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN`, Min. Zoom 5, Max. Zoom 16, tile resolution 512.
    3. The layer shows the raw tiles (encoded color). Elevations follow from the formula in “Encoding” (`(R*65536 + G*256 + B)*0.1 - 10000`), for example in the Raster Calculator on the three bands.
  </TabItem>
  <TabItem label="Python">
    <Code lang="python" code={`# pip install pmtiles pillow numpy requests
from PIL import Image
from pmtiles.reader import Reader

URL = "${url}?access_token=YOUR_TOKEN"

def get_bytes(offset, length):
    r = requests.get(URL, headers={"Range": f"bytes={offset}-{offset + length - 1}"}, timeout=30)
    r.raise_for_status()
    return r.content

reader = Reader(get_bytes)
tile = reader.get(12, 2192, 1439)          # z, x, y (Grossglockner)
rgb = np.asarray(Image.open(io.BytesIO(tile)).convert("RGB")).astype(np.uint32)
hoehe = -10000 + (rgb[..., 0] * 65536 + rgb[..., 1] * 256 + rgb[..., 2]) * 0.1
print(hoehe.shape, hoehe.min(), hoehe.max())`} />
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`# one tile via the XYZ endpoint
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/terrain-rgb/12/2192/1439.webp

# whole file, resumable
curl -C - -O "${url}?access_token=YOUR_TOKEN"`} />
  </TabItem>
</Tabs>

## Encoding

```
height_m = -10000 + (R · 65536 + G · 256 + B) · 0.1
```

- Theoretical value range −10000 m to +1,667,721.5 m in steps of 0.1 m; in Austria the values lie between about 115 m (Lake Neusiedl) and about 3,798 m (Grossglockner).
- Values are rounded to 0.1 m when written (round half to even at exactly .5, see Rust code above); values outside the range are clamped.
- WebP **lossless**: lossy compression would directly corrupt the encoded elevations. Do not use alpha information; read the RGB bands unchanged (no color management, no premultiplied alpha).
- 512 × 512 pixels, **no border** (no 1 px skirt). In MapLibre specify `tileSize: 512` and `encoding: "mapbox"`.

In detail: [Terrain-RGB](/reference/terrain-rgb/).

## Vertical resolution and accuracy

| | Value | Meaning |
|---|---|---|
| Vertical resolution | **0.1 m** | smallest height step of the encoding |
| Deviation from source | **≤ 0.05 m** | rounding during encoding, otherwise unchanged (lossless) |
| Vertical accuracy of the source | **± 0.50 m** | stated by BEV for the ALS-DTM; larger deviations are possible in high mountains |
| Height reference | EVRF2000 Austria | orthometric heights (EPSG:9274), as in the source |

The encoding is five times finer than the accuracy of the survey, so it does not limit quality.

In the overview levels z15 to z5 each pixel is the mean of the finer level. The vertical resolution stays 0.1 m, but small peaks and summits are smoothed. For exact point heights read z16 or use the [elevation API](/api/elevation/).

## Horizontal resolution by zoom

At 47.5° N, 512 px tiles, Web Mercator:

<Aufloesung lang="en" />

The data come from a 1 m raster. z16 (about 0.8 m per pixel) is the finest level and preserves the resolution of the source.
There are no tiles above z16: from z17 MapLibre scales up the z16 tile (set `maxzoom: 16`).

## Source and currency

The DTM is derived from the BEV ALS-DTM 1 m (airborne laser scanning, terrain only). The reference date is in the specifications and the file name; a file never changes, a new release is a new file.

- Source: BEV (Austrian Federal Office of Metrology and Surveying), reference date 15 Sep 2025.

## Notes

- Quantization 0.1 m. The deviation from the source is within the rounding range.
- Vertical accuracy per BEV ± 0.50 m, see [Vertical resolution and accuracy](#vertical-resolution-and-accuracy).
- The DTM shows the terrain without buildings, trees and bridges. For the surface there is the [DSM](/tilesets/dsm/).
- Coverage is the rectangle 9.53–17.17° E, 46.37–49.02° N. Outside Austria and in gaps the value is a smooth continuation of the terrain, not a measurement.

## Attribution

Required under CC BY 4.0. Text:

```
Relief: BEV – ALS-DTM 1 m (reference date 15 Sep 2025), CC BY 4.0
```

HTML (as in the PMTiles metadata; MapLibre picks it up automatically from the tileset):

<Code lang="html" code={data.tilesets.relief.metadata.attribution} />

More: [License](/reference/attribution/).
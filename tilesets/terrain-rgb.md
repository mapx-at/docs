# DTM terrain model tileset (Terrain-RGB)

Source: https://docs.mapx.at/tilesets/terrain-rgb/

> Digital terrain model of Austria as Terrain-RGB tiles (Mapbox encoding, 0.1 m), z5 to z16, from the BEV ALS-DTM 1 m.

The DTM is an elevation tile set for all of Austria: ground height without buildings and vegetation, encoded as color (Terrain-RGB). It works directly as `raster-dem` for hillshading and 3D terrain in MapLibre, Mapbox GL and compatible clients.

**Use cases**

- **Hiking and outdoor maps**: Hillshade as a map background up to z16
- **3D terrain**: Terrain and flight views with orthophoto (https://docs.mapx.at/guides/3d-terrain/)
- **Elevation profiles**: Ascent and descent along a route (https://docs.mapx.at/guides/elevation-profile/)
- **Slope and aspect analysis**: Gradient and orientation from the elevation values
- **QGIS and analysis**: Elevation as raster in desktop GIS (https://docs.mapx.at/guides/qgis/)

## Specifications

**Tileset `terrain-rgb`**

| Property | Value |
|---|---|
| Reference date | `2025-09-15` |
| Format | `webp, encoding mapbox, 512 px` |
| Zoom levels | `z5–z16` |
| Bounds (WGS84) | `9.5306 / 46.3697 / 17.1661 / 49.0235 (W/S/O/N)` |
| Vertical resolution | `0.1 m (Terrain-RGB)` |
| Vertical accuracy (source) | `± 0.50 m per BEV, larger in high mountains` |
| Height reference | `EVRF2000 Austria, orthometric (EPSG:9274)` |
| License | `CC BY 4.0` |

## Live map

Clicking reads the tile of the current zoom level (z16 at most) via range request, decodes the pixel at the click point with the
formula under “Encoding” and shows elevation and RGB value. With 3D enabled, the value of `queryTerrainElevation` is shown as well.

## Add to your map

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  center: [12.694, 47.074], zoom: 11, pitch: 60,
  style: {
    version: 8,
    sources: { dem: { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512, encoding: "mapbox" } },
    terrain: { source: "dem", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dem" }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
map.on("click", (e) => console.log(map.queryTerrainElevation(e.lngLat)));
```
Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

1. In the Browser panel, right-click **XYZ Tiles**, **New Connection**.
2. URL `https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN`, Min. Zoom 5, Max. Zoom 16, tile resolution 512.
3. The layer shows the raw tiles (encoded color). Elevations follow from the formula in “Encoding” (`(R*65536 + G*256 + B)*0.1 - 10000`), for example in the Raster Calculator on the three bands.

  

**Python**

```python
# pip install pillow numpy requests
import io
import numpy as np
import requests
from PIL import Image

z, x, y = 12, 2192, 1439                   # Grossglockner
r = requests.get(f"https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp",
                 headers={"Authorization": "Bearer YOUR_TOKEN"}, timeout=30)
r.raise_for_status()
tile = r.content
rgb = np.asarray(Image.open(io.BytesIO(tile)).convert("RGB")).astype(np.uint32)
hoehe = -10000 + (rgb[..., 0] * 65536 + rgb[..., 1] * 256 + rgb[..., 2]) * 0.1
print(hoehe.shape, hoehe.min(), hoehe.max())
```

  

**curl**

```bash
# one tile via the XYZ endpoint
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/terrain-rgb/12/2192/1439.webp
```

## Encoding

```
height_m = -10000 + (R · 65536 + G · 256 + B) · 0.1
```

- Theoretical value range −10000 m to +1,667,721.5 m in steps of 0.1 m; in Austria the values lie between about 115 m (Lake Neusiedl) and about 3,798 m (Grossglockner).
- Values are rounded to 0.1 m when written (round half to even at exactly .5, see Rust code above); values outside the range are clamped.
- WebP **lossless**: lossy compression would directly corrupt the encoded elevations. Do not use alpha information; read the RGB bands unchanged (no color management, no premultiplied alpha).
- 512 × 512 pixels, **no border** (no 1 px skirt). In MapLibre specify `tileSize: 512` and `encoding: "mapbox"`.

In detail: [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md).

## Vertical resolution and accuracy

| | Value | Meaning |
|---|---|---|
| Vertical resolution | **0.1 m** | smallest height step of the encoding |
| Deviation from source | **≤ 0.05 m** | rounding during encoding, otherwise unchanged (lossless) |
| Vertical accuracy of the source | **± 0.50 m** | stated by BEV for the ALS-DTM; larger deviations are possible in high mountains |
| Height reference | EVRF2000 Austria | orthometric heights (EPSG:9274), as in the source |

The encoding is five times finer than the accuracy of the survey, so it does not limit quality.

In the overview levels z15 to z5 each pixel is the mean of the finer level. The vertical resolution stays 0.1 m, but small peaks and summits are smoothed. For exact point heights read z16 or use the [elevation API](https://docs.mapx.at/api/elevation.md).

## Horizontal resolution by zoom

At 47.5° N, 512 px tiles, Web Mercator:

| Zoom | m per pixel | Tile edge (km) | Note |
|---|---|---|---|
| z5 | 1,652 | 846 |  |
| z6 | 826 | 423 |  |
| z7 | 413 | 212 |  |
| z8 | 207 | 106 |  |
| z9 | 103 | 53 |  |
| z10 | 52 | 26 |  |
| z11 | 26 | 13 |  |
| z12 | 13 | 6.61 |  |
| z13 | 6.46 | 3.30 |  |
| z14 | 3.23 | 1.65 |  |
| z15 | 1.61 | 0.83 |  |
| z16 | 0.81 | 0.41 | highest stored level (≈ 1 m source) |
| z17 | 0.40 | 0.21 | overzoom (client) |

The data come from a 1 m raster. z16 (about 0.8 m per pixel) is the finest level and preserves the resolution of the source.
There are no tiles above z16: from z17 MapLibre scales up the z16 tile (set `maxzoom: 16`).

## Source and currency

The DTM is derived from the BEV ALS-DTM 1 m (airborne laser scanning, terrain only). The reference date is in the specifications and the file name; a file never changes, a new release is a new file.

- Source: BEV (Austrian Federal Office of Metrology and Surveying), reference date 15 Sep 2025.

## Notes

- Quantization 0.1 m. The deviation from the source is within the rounding range (at most 0.05 m).
- Vertical accuracy per BEV ± 0.50 m, see [Vertical resolution and accuracy](#vertical-resolution-and-accuracy).
- The DTM shows the terrain without buildings, trees and bridges. For the surface there is the [DSM](https://docs.mapx.at/tilesets/dsm.md).
- Coverage is the rectangle 9.53–17.17° E, 46.37–49.02° N. Outside Austria and in gaps the value is a smooth continuation of the terrain, not a measurement.

## Attribution

Required under CC BY 4.0. Text:

```
mapx.at | Relief: BEV – ALS-DTM 1 m (reference date 15 Sep 2025), CC BY 4.0
```

HTML (as in the TileJSON; MapLibre picks it up automatically from the TileJSON):

```html
mapx.at | Relief: <a href="https://data.bev.gv.at/geonetwork/srv/ger/catalog.search#/metadata/5ce253fc-b7c5-4362-97af-6556c18a45d9">BEV – ALS-DTM 1 m</a> (Stichtag 15.09.2025), CC BY 4.0
```

More: [License](https://docs.mapx.at/reference/attribution.md).

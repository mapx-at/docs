# Terrain-RGB

How mapx encodes elevation as color: value range, rounding and decoding functions.

Terrain-RGB stores one elevation per pixel in the three color channels of an image. This lets elevation tiles be served in ordinary
image formats, and clients such as MapLibre compute shading and 3D terrain from them.

## Formula

```
height_m = -10000 + (R · 65536 + G · 256 + B) · 0.1
```

R is the most significant byte. The value `q = R·65536 + G·256 + B` lies between 0 and 16,777,215, so the elevation lies between
**−10,000 m and +1,667,721.5 m** in steps of 0.1 m.

## Encoding

1. Multiply the elevation by 10: `(h + 10000) · 10`.
2. Round to the nearest integer (mapx: round half to even at exactly .5).
3. Clamp to 0 to 16,777,215.
4. Split into three bytes: `R = q >> 16`, `G = (q >> 8) & 255`, `B = q & 255`.

## Decoding

<Code lang="js" code={`const hoehe = (r, g, b) => -10000 + (r * 65536 + g * 256 + b) * 0.1;`} />
<Code lang="python" code={`hoehe = -10000 + (r.astype("uint32") * 65536 + g * 256 + b) * 0.1  # r, g, b als numpy-Arrays`} />
<Code lang="rust" code={`fn hoehe(r: u8, g: u8, b: u8) -> f64 {
    -10000.0 + (u32::from(r) * 65536 + u32::from(g) * 256 + u32::from(b)) as f64 * 0.1
}`} />

## What to watch for

- **Read the image format losslessly.** mapx serves lossless WebP. Any lossy intermediate step (JPEG, color profile, premultiplied alpha, scaling while decoding) distorts the elevation.
- **Do not interpolate before decoding.** Blending RGB values does not give a blended elevation. Decode first, then average. This is also how the overview levels are made.
- **Mapbox-compatible.** `encoding: "mapbox"` in MapLibre. Terrarium (`R·256 + G + B/256 − 32768`) is a different encoding and is not served by mapx.
- **512 pixels, no border.** `tileSize: 512`.

Application: [DTM terrain model](/en/daten/dtm/).
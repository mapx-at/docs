# Fonts

Glyphs as protobuf for MapLibre and Mapbox GL, with several fonts combined into one font stack.

The fonts for labels, in the shape of the Mapbox Fonts API. No key, CORS open.

**GET https://api.mapx.at/v1/fonts**

List of available fonts, e.g. `Noto Sans Regular`, `Noto Sans Bold`, `Open Sans Regular`.

**GET https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf**

<ParamTable params={[
  { name: "fontstack", type: "string", required: true, desc: "One font or several, comma-separated (spaces as <code>%20</code>), at most 8. For each character the first font that has it wins (fallback as in Mapbox)." },
  { name: "range", type: "string", required: true, desc: "<code>0-255</code>, <code>256-511</code>, …: 256 characters from a multiple of 256, up to <code>65280-65535</code>." },
]} />

The response is `application/x-protobuf` with `ETag` and `Cache-Control: public, max-age=86400`. Unknown font: `404`, invalid range or name: `400`.

## In a style

MapLibre and Mapbox GL request the ranges themselves when the style has a `glyphs` field. The styles from <a href="/en/api/styles/">/v1/styles</a> already contain it:

```json
{
  "glyphs": "https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf",
  "layers": [{ "type": "symbol", "layout": { "text-font": ["Noto Sans Regular", "Open Sans Regular"], "text-field": "{name}" } }]
}
```

`text-font` with two entries gives the font stack `Noto Sans Regular,Open Sans Regular`.
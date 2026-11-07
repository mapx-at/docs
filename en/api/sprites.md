# Sprites

Icon atlases (sprite index as JSON and image as PNG) for the styles, normal and @2x, plus SDF icons.

The map icons, in the shape of the Mapbox Sprites API. No key, CORS open.

**GET https://api.mapx.at/v1/styles/{id}/sprite[@2x].{json,png}**

**GET https://api.mapx.at/v1/styles/{id}/sprite-sdf[@2x].{json,png}**

**GET https://api.mapx.at/v1/sprites**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Style from <a href=\"/en/api/styles/\">/v1/styles</a> (<code>karte</code>, <code>luftbild</code>, …). Unknown style: <code>404</code>." },
  { name: "@2x", type: "suffix", desc: "Double resolution for high pixel density screens." },
  { name: "json / png", type: "suffix", required: true, desc: "<code>.json</code> is the index (name, position, size, <code>pixelRatio</code>), <code>.png</code> the atlas." },
]} />

`sprite` holds the coloured icons, `sprite-sdf` single-colour SDF icons that the style tints with `icon-color`. Currently all styles use the same sprites; `/v1/sprites` lists them, `/v1/sprites/{file}` serves them without a style. Responses carry `ETag` and `Cache-Control: public, max-age=86400`.

## In a style

The styles from <a href="/en/api/styles/">/v1/styles</a> point at these addresses; MapLibre appends `.json`, `.png` and `@2x` itself:

```json
{
  "sprite": [
    { "id": "default", "url": "https://api.mapx.at/v1/styles/karte/sprite" },
    { "id": "sdf", "url": "https://api.mapx.at/v1/styles/karte/sprite-sdf" }
  ]
}
```

Icons of the second sprite are called `sdf:<name>` in the style.
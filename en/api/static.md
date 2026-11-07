# Static Images

Map images as PNG or WebP for aerial imagery and terrain, with pin, line and GeoJSON overlays.

The endpoint renders a map view as an image, in the shape of the Mapbox Static Images API. Suited for thumbnails, e-mails, PDF reports and pages without JavaScript. No key, open CORS.

**GET https://api.mapx.at/v1/static/{style}/{view}/{size}**

**GET https://api.mapx.at/v1/static/{style}/{overlay}/{view}/{size}**

<ParamTable params={[
  { name: "style", type: "string", required: true, desc: "Base of the image", values: [
    { v: "luftbild", d: "BEV orthophoto, z5 to z18" },
    { v: "gelaende", d: "Elevation tints with hillshade from the DTM, z5 to z16" },
  ] },
  { name: "overlay", type: "string", desc: "Comma-separated list of overlays (see below), at most 50 with 20000 vertices in total." },
  { name: "view", type: "string", required: true, desc: "<code>lon,lat,zoom[,bearing]</code> (rotation in degrees, pitch only 0) or <code>auto</code> (only with overlays, fits the view to them)." },
  { name: "size", type: "string", required: true, desc: "<code>{width}x{height}[@2x][.png|.webp]</code>, each 1 to 1280 pixels; with <code>@2x</code> up to 2560 pixels. Default is PNG." },
]} />

> The vector style `karte` is not rendered and returns `400`. Use MapLibre with [Styles](/en/api/styles/) for it.

## Overlays

| Overlay | Form | Note |
|---|---|---|
| Pin | `pin-{s\|m\|l}[-label]+{hex}(lon,lat)` | size and colour; the label is not drawn |
| Line, area | `path-{width}[+{stroke colour}[-{opacity}][+{fill colour}[-{opacity}]]]({polyline})` | polyline with precision 5, URL-encoded |
| GeoJSON | `geojson({URL-encoded GeoJSON})` | simplestyle properties: `stroke`, `stroke-width`, `stroke-opacity`, `fill`, `fill-opacity`, `marker-color`, `marker-size` |

## Examples

<Code lang="bash" code={`# Aerial imagery with a red pin, 600 x 400, PNG
curl -o map.png "https://api.mapx.at/v1/static/luftbild/pin-s+ff0000(15.33,48.22)/15.33,48.22,15/600x400.png"

# Terrain image, WebP at double resolution
curl -o terrain.webp "https://api.mapx.at/v1/static/gelaende/12.69,47.07,12/800x600@2x.webp"

# View fitted automatically around two pins
curl -o auto.png "https://api.mapx.at/v1/static/luftbild/pin-m+0000ff(15.33,48.22),pin-m+0000ff(15.35,48.24)/auto/600x400.png"`} />

In HTML an `img` element is enough:

<Code lang="html" code={`<img src="https://api.mapx.at/v1/static/luftbild/15.33,48.22,15/600x400.webp" alt="Aerial view of Krummnußbaum" width="600" height="400">`} />

## Response and caching

`200` with `image/png` or `image/webp`. The response carries `Cache-Control` and `ETag`; with `If-None-Match` an unchanged image returns `304`.

## Errors

Errors are returned as `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)): `400` invalid input (for example style `karte`, size above 1280, invalid overlay), `404` unknown path, `429` rate limit exceeded (honour `Retry-After`), `503` temporarily unavailable. General: [Errors and limits](/en/api/fehler/). Machine-readable: [openapi.json](https://api.mapx.at/v1/openapi.json).

For attribution see [License](/en/lizenz/): images with orthophoto need the BEV notice.
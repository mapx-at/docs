# Static images

Render map images as PNG or WebP with pins, paths and GeoJSON, for reports, emails and web pages.

A static image is one HTTP request that returns a picture. Use it where an interactive map is not possible: PDFs, emails, listings, social previews. The token goes into the URL as `access_token`, because an image tag cannot send a header. For web pages use a public token (`mx_pk_…`) with your site as allowed origin; build images for e-mails and PDFs on your server with a secret token (`mx_sk_…`) so the secret does not end up in the message.

```html
<img alt="Vienna city center" width="600" height="400"
     src="https://api.mapx.at/v1/static/streets/16.3725,48.2085,14/600x400.png?access_token=YOUR_TOKEN">
```

## URL format

```
/v1/static/{style}/{view}/{width}x{height}[@2x][.png|.webp]
/v1/static/{style}/{overlay}/{view}/{width}x{height}...
```

| Part | Values |
|---|---|
| `style` | `streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors` |
| `view` | `lon,lat,zoom[,bearing]` or `auto` (fits the overlays) |
| size | width and height 1 to 1280, up to 2560 with `@2x` |
| format | `.png` or `.webp` |

Images are in the `heavy` zone (30 per minute). They are cacheable: send `If-None-Match` and the API answers `304`. Cache images on your side and do not request them per page view.

## Overlays

Overlays come after the style, separated by commas (up to 50).

| Overlay | Example |
|---|---|
| Pin | `pin-s+ff0000(16.3725,48.2085)` (size `s`, `m` or `l`, color as hex) |
| Path | `path-4+1d5bd8-0.8(encodedPolyline)` (width, stroke, opacity; polyline precision 5, URL-encoded) |
| GeoJSON | `geojson({...})` with simplestyle properties (URL-encoded) |

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/static/streets/pin-s+ff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400.png" -o vienna.png
```

## Fit a route automatically

With `auto` instead of the view, the image shows all overlays.

```js
const dir = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?geometries=polyline&overview=simplified&access_token=YOUR_TOKEN";
const { routes } = await (await fetch(dir)).json();
const path = encodeURIComponent(routes[0].geometry); // encoded polyline, precision 5
const url = `https://api.mapx.at/v1/static/outdoors/path-5+e11d48(${path})/auto/800x500@2x.webp?access_token=YOUR_TOKEN`;
```

The route comes from [Directions](/guides/navigation/) with `geometries=polyline`.

## Aerial image for a listing

`satellite` shows the orthophoto, `outdoors` adds contours and trails.

```
https://api.mapx.at/v1/static/satellite/pin-m+ff0000(15.3294,48.2272)/15.3294,48.2272,17/800x600.webp?access_token=YOUR_TOKEN
```

Show the data credit next to the image, see [Attribution](/reference/attribution/).

## Raster tiles of a style

For clients that only read raster tiles (Leaflet, QGIS), Static Tiles render a style as 256 or 512 px tiles: `/v1/styles/{id}/tiles/{256|512}/{z}/{x}/{y}`.

## Next

- [Static Images API](/api/static-images/)
- [Maps](/guides/maps/): the same styles interactively
- [Parcels](/guides/parcels/): parcel boundary as GeoJSON overlay
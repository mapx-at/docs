# StaticOptions

## Properties

### bearing?

> `optional` **bearing**: `number`

***

### center?

> `optional` **center**: [`LL`](/reference/sdk/type-aliases/ll/)

Mittelpunkt und Zoom; ohne beides `auto` (nur mit Overlays).

***

### format?

> `optional` **format**: `"png"` \| `"webp"`

***

### height

> **height**: `number`

***

### overlays?

> `optional` **overlays**: `string`[]

Overlays: fertige Strings (`pin-s+e11d48(16.37,48.2)`) oder Helfer `pin()`, `geojsonOverlay()`.

***

### retina?

> `optional` **retina**: `boolean`

Doppelte Auflösung.

***

### style?

> `optional` **style**: `string`

`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `satellite-streets`, `outdoors`. Standard `streets`.

***

### width

> **width**: `number`

Bildgröße in Pixel, je 1 bis 1280.

***

### zoom?

> `optional` **zoom**: `number`


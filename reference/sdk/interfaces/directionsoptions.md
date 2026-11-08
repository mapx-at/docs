# DirectionsOptions

mapx-gl DirectionsOptions

## Extends

- [`Common`](/reference/sdk/interfaces/common/)

## Properties

### alternatives?

> `optional` **alternatives**: `number` \| `boolean`

`true` (bis 2) oder 0 bis 3 Alternativen, nur mit genau 2 Punkten.

***

### annotations?

> `optional` **annotations**: (`"duration"` \| `"distance"` \| `"speed"` \| `"maxspeed"` \| `"elevation"`)[]

***

### exclude?

> `optional` **exclude**: (`"motorway"` \| `"toll"`)[]

Autobahn und/oder mautpflichtige Abschnitte meiden (nur Profil `driving`).

***

### geometries?

> `optional` **geometries**: `"geojson"` \| `"polyline"` \| `"polyline6"`

***

### incidents?

> `optional` **incidents**: `boolean`

***

### language?

> `optional` **language**: `"de"` \| `"en"`

***

### overview?

> `optional` **overview**: `"full"` \| `"simplified"` \| `"false"`

***

### signal?

> `optional` **signal**: `AbortSignal`

#### Inherited from

[`Common`](/reference/sdk/interfaces/common/).[`signal`](/reference/sdk/interfaces/common/#signal)

***

### steps?

> `optional` **steps**: `boolean`

***

### truck?

> `optional` **truck**: `object`

Nur `driving-truck`: Gesamtgewicht t (0,5–60), Höhe m (1,5–5), Breite m (1–3,5), Länge m (2–40).

#### height?

> `optional` **height**: `number`

#### length?

> `optional` **length**: `number`

#### type?

> `optional` **type**: `"semi"` \| `"rigid"` \| `"trailer"`

#### weight?

> `optional` **weight**: `number`

#### width?

> `optional` **width**: `number`
# Dossier

mapx-gl Dossier

## Extends

- `Feature`\<`GeoJSON.Polygon` \| `GeoJSON.MultiPolygon`\>

## Properties

### links

> **links**: `object`

#### self

> **self**: `string`

#### static\_image

> **static\_image**: `string`

#### transit\_stops?

> `optional` **transit\_stops**: `string`

***

### properties

> **properties**: `object`

Properties associated with this feature.

#### addresses

> **addresses**: `object`[]

#### area\_ellipsoidal\_m2

> **area\_ellipsoidal\_m2**: `number`

#### area\_m2

> **area\_m2**: `number`

#### buildings

> **buildings**: `object`

#### buildings.count

> **count**: `number`

#### buildings.footprint\_m2

> **footprint\_m2**: `number`

#### buildings.height\_max\_m

> **height\_max\_m**: `null` \| `number`

#### buildings.height\_mean\_m

> **height\_mean\_m**: `null` \| `number`

#### buildings.items

> **items**: `object`[]

#### cadastral\_municipality

> **cadastral\_municipality**: `object`

#### cadastral\_municipality.id

> **id**: `string`

#### cadastral\_municipality.name

> **name**: `null` \| `string`

#### centroid

> **centroid**: [`LL`](/reference/sdk/type-aliases/ll/)

#### district

> **district**: `object`

#### district.id

> **id**: `null` \| `string`

#### district.name

> **name**: `null` \| `string`

#### elevation

> **elevation**: `null` \| `object`

#### is\_building\_land

> **is\_building\_land**: `boolean`

#### land\_use

> **land\_use**: `object`[]

#### municipality

> **municipality**: `object`

#### municipality.id

> **id**: `null` \| `string`

#### municipality.name

> **name**: `null` \| `string`

#### nearest\_transit\_stop

> **nearest\_transit\_stop**: `null` \| `object`

#### orthophoto

> **orthophoto**: `null` \| `object`

#### parcel\_id

> **parcel\_id**: `string`

#### parcel\_number

> **parcel\_number**: `string`

#### reference\_date

> **reference\_date**: `string`

#### state

> **state**: `object`

#### state.id

> **id**: `null` \| `string`

#### state.name

> **name**: `null` \| `string`

#### Overrides

`GeoJSON.Feature.properties`
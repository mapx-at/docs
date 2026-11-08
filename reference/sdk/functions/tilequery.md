# tilequery

mapx-gl tilequery

> **tilequery**(`tilesets`, `point`, `o`): `Promise`\<`GeoJSON.FeatureCollection`\>

Vektorkachel-Abfrage: nächste Objekte um einen Punkt (`radius` bis 2000 m). `tilesets` z. B. `['streets','cadastre']`.

## Parameters

• **tilesets**: `string`[]

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`TilequeryOptions`](/reference/sdk/interfaces/tilequeryoptions/) = `{}`

## Returns

`Promise`\<`GeoJSON.FeatureCollection`\>
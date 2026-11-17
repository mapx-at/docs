# tilequery

Source: https://docs.mapx.at/reference/sdk/functions/tilequery/

> mapx-gl tilequery

> **tilequery**(`tilesets`, `point`, `o`): `Promise`\<`GeoJSON.FeatureCollection`\>

Vektorkachel-Abfrage: nächste Objekte um einen Punkt (`radius` bis 2000 m). `tilesets` z. B. `['streets','cadastre']`.

## Parameters

• **tilesets**: `string`[]

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`TilequeryOptions`](https://docs.mapx.at/reference/sdk/interfaces/tilequeryoptions.md) = `{}`

## Returns

`Promise`\<`GeoJSON.FeatureCollection`\>

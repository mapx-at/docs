# elevation

Source: https://docs.mapx.at/reference/sdk/functions/elevation/

> mapx-gl elevation

> **elevation**(`points`, `model`): `Promise`\<`number` \| `null` \| (`number` \| `null`)[]\>

Höhe (m) für einen oder mehrere Punkte; null außerhalb der Daten. `dsm` = Oberflächenmodell.

## Parameters

• **points**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md) \| [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

## Returns

`Promise`\<`number` \| `null` \| (`number` \| `null`)[]\>

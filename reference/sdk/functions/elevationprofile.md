# elevationProfile

Source: https://docs.mapx.at/reference/sdk/functions/elevationprofile/

> mapx-gl elevationProfile

> **elevationProfile**(`line`, `samples`, `model`, `signal`?): `Promise`\<[`ElevationProfile`](https://docs.mapx.at/reference/sdk/interfaces/elevationprofile.md)\>

Höhenprofil entlang einer Linie (2 bis 512 Stützpunkte).

## Parameters

• **line**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **samples**: `number` = `100`

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

• **signal?**: `AbortSignal`

## Returns

`Promise`\<[`ElevationProfile`](https://docs.mapx.at/reference/sdk/interfaces/elevationprofile.md)\>

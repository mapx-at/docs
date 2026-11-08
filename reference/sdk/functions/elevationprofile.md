# elevationProfile

> **elevationProfile**(`line`, `samples`, `model`, `signal`?): `Promise`\<[`ElevationProfile`](/reference/sdk/interfaces/elevationprofile/)\>

Höhenprofil entlang einer Linie (2 bis 512 Stützpunkte).

## Parameters

• **line**: [`LL`](/reference/sdk/type-aliases/ll/)[]

• **samples**: `number` = `100`

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

• **signal?**: `AbortSignal`

## Returns

`Promise`\<[`ElevationProfile`](/reference/sdk/interfaces/elevationprofile/)\>


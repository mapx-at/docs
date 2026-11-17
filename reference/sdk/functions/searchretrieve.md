# searchRetrieve

Source: https://docs.mapx.at/reference/sdk/functions/searchretrieve/

> mapx-gl searchRetrieve

> **searchRetrieve**(`id`, `o`): `Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Point`\>\>

Search Box, Schritt 2: Koordinate und Details zu einem Vorschlag (`mapbox_id`).

## Parameters

• **id**: `string`

• **o**: `Pick`\<[`SearchOptions`](https://docs.mapx.at/reference/sdk/interfaces/searchoptions.md), `"proximity"` \| `"sessionToken"` \| `"signal"`\> = `{}`

## Returns

`Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Point`\>\>

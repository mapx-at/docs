# weatherForecast

Source: https://docs.mapx.at/reference/sdk/functions/weatherforecast/

> mapx-gl weatherForecast

> **weatherForecast**(`point`, `o`): `Promise`\<`object`\>

Stundenvorhersage (AROME 1 km) für einen Punkt.

## Parameters

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`Common`](https://docs.mapx.at/reference/sdk/interfaces/common.md) & `object` = `{}`

## Returns

`Promise`\<`object`\>

### hours

> **hours**: [`WeatherHour`](https://docs.mapx.at/reference/sdk/interfaces/weatherhour.md)[]

### model

> **model**: `string`

### reference\_time

> **reference\_time**: `string`

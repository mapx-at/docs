# weatherForecast

> **weatherForecast**(`point`, `o`): `Promise`\<`object`\>

Stundenvorhersage (AROME 1 km) für einen Punkt.

## Parameters

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`Common`](/reference/sdk/interfaces/common/) & `object` = `{}`

## Returns

`Promise`\<`object`\>

### hours

> **hours**: [`WeatherHour`](/reference/sdk/interfaces/weatherhour/)[]

### model

> **model**: `string`

### reference\_time

> **reference\_time**: `string`


# directions

Source: https://docs.mapx.at/reference/sdk/functions/directions/

> mapx-gl directions

> **directions**(`profile`, `waypoints`, `o`): `Promise`\<[`DirectionsResponse`](https://docs.mapx.at/reference/sdk/interfaces/directionsresponse.md)\>

Routing (Mapbox-Directions-Antwort, Geometrie als GeoJSON). Profile: driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking.

## Parameters

• **profile**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)

• **waypoints**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **o**: [`DirectionsOptions`](https://docs.mapx.at/reference/sdk/interfaces/directionsoptions.md) = `{}`

## Returns

`Promise`\<[`DirectionsResponse`](https://docs.mapx.at/reference/sdk/interfaces/directionsresponse.md)\>

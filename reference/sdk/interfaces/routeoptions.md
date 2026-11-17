# RouteOptions

Source: https://docs.mapx.at/reference/sdk/interfaces/routeoptions/

> mapx-gl RouteOptions

## Properties

### onError()?

> `optional` **onError**: (`e`) => `void`

#### Parameters

• **e**: [`MapxError`](https://docs.mapx.at/reference/sdk/classes/mapxerror.md)

#### Returns

`void`

***

### onRoute()?

> `optional` **onRoute**: (`r`, `request`) => `void`

#### Parameters

• **r**: [`DirectionsResponse`](https://docs.mapx.at/reference/sdk/interfaces/directionsresponse.md)

• **request**

• **request.options**: [`DirectionsOptions`](https://docs.mapx.at/reference/sdk/interfaces/directionsoptions.md)

• **request.profile**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)

• **request.waypoints**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

#### Returns

`void`

***

### options?

> `optional` **options**: `boolean`

Maut-/Autobahn-Schalter und Lkw-Maße anzeigen (Standard `true`).

***

### profile?

> `optional` **profile**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)

***

### profiles?

> `optional` **profiles**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)[]

Angebotene Profile; Standard alle sechs.

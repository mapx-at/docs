# RouteOptions

## Properties

### onError()?

> `optional` **onError**: (`e`) => `void`

#### Parameters

• **e**: [`MapxError`](/reference/sdk/classes/mapxerror/)

#### Returns

`void`

***

### onRoute()?

> `optional` **onRoute**: (`r`, `request`) => `void`

#### Parameters

• **r**: [`DirectionsResponse`](/reference/sdk/interfaces/directionsresponse/)

• **request**

• **request.options**: [`DirectionsOptions`](/reference/sdk/interfaces/directionsoptions/)

• **request.profile**: [`Profile`](/reference/sdk/type-aliases/profile/)

• **request.waypoints**: [`LL`](/reference/sdk/type-aliases/ll/)[]

#### Returns

`void`

***

### options?

> `optional` **options**: `boolean`

Maut-/Autobahn-Schalter und Lkw-Maße anzeigen (Standard `true`).

***

### profile?

> `optional` **profile**: [`Profile`](/reference/sdk/type-aliases/profile/)

***

### profiles?

> `optional` **profiles**: [`Profile`](/reference/sdk/type-aliases/profile/)[]

Angebotene Profile; Standard alle sechs.


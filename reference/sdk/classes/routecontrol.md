# RouteControl

Routenplaner: Start/Ziel per Klick, Profile inkl. Lkw und MTB, Maut/Autobahn meiden, Lkw-Maße. Ergebnis in `last` und `onRoute`.

## Implements

- `IControl`

## Constructors

### new RouteControl()

> **new RouteControl**(`o`): [`RouteControl`](/reference/sdk/classes/routecontrol/)

#### Parameters

• **o**: [`RouteOptions`](/reference/sdk/interfaces/routeoptions/) = `{}`

#### Returns

[`RouteControl`](/reference/sdk/classes/routecontrol/)

## Properties

### last

> **last**: `null` \| [`DirectionsResponse`](/reference/sdk/interfaces/directionsresponse/) = `null`

## Methods

### onAdd()

> **onAdd**(`map`): `HTMLElement`

Register a control on the map and give it a chance to register event listeners
and resources. This method is called by Map#addControl
internally.

#### Parameters

• **map**: `Map$1`

the Map this control will be added to

#### Returns

`HTMLElement`

The control's container element. This should
be created by the control and returned by onAdd without being attached
to the DOM: the map will insert the control's element into the DOM
as necessary.

#### Implementation of

`IControl.onAdd`

***

### onRemove()

> **onRemove**(): `void`

Unregister a control on the map and give it a chance to detach event listeners
and resources. This method is called by Map#removeControl
internally.

#### Returns

`void`

#### Implementation of

`IControl.onRemove`

***

### reset()

> **reset**(): `void`

#### Returns

`void`

***

### setWaypoints()

> **setWaypoints**(`from`, `to`): `Promise`\<`void`\>

Start und Ziel setzen und sofort rechnen.

#### Parameters

• **from**: [`LL`](/reference/sdk/type-aliases/ll/)

• **to**: [`LL`](/reference/sdk/type-aliases/ll/)

#### Returns

`Promise`\<`void`\>


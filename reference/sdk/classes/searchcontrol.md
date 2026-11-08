# SearchControl

mapx-gl SearchControl

Suchfeld über /v1/geocode; Auswahl fliegt zum Treffer.

## Implements

- `IControl`

## Constructors

### new SearchControl()

> **new SearchControl**(`o`): [`SearchControl`](/reference/sdk/classes/searchcontrol/)

#### Parameters

• **o**: `SearchOptions` = `{}`

#### Returns

[`SearchControl`](/reference/sdk/classes/searchcontrol/)

## Methods

### onAdd()

> **onAdd**(`map`): `HTMLDivElement`

Register a control on the map and give it a chance to register event listeners
and resources. This method is called by Map#addControl
internally.

#### Parameters

• **map**: `Map$1`

the Map this control will be added to

#### Returns

`HTMLDivElement`

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
# StyleSwitcherControl

mapx-gl StyleSwitcherControl

Dropdown zum Umschalten der Styles (Kurznamen aus /v1/styles). Gelände bleibt erhalten.

## Implements

- `IControl`

## Constructors

### new StyleSwitcherControl()

> **new StyleSwitcherControl**(`o`): [`StyleSwitcherControl`](/reference/sdk/classes/styleswitchercontrol/)

#### Parameters

• **o**: `StyleSwitcherOptions` = `{}`

#### Returns

[`StyleSwitcherControl`](/reference/sdk/classes/styleswitchercontrol/)

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
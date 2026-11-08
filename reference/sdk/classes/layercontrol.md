# LayerControl

Ebenen-Schalter: Gruppen und Overlays kommen aus den Style-Metadaten (`mapx:groups`, `mapx:overlays`), keine Liste im Code nötig.

## Implements

- `IControl`

## Constructors

### new LayerControl()

> **new LayerControl**(`o`): [`LayerControl`](/reference/sdk/classes/layercontrol/)

#### Parameters

• **o**: [`LayerControlOptions`](/reference/sdk/interfaces/layercontroloptions/) = `{}`

#### Returns

[`LayerControl`](/reference/sdk/classes/layercontrol/)

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

### setGroupVisible()

> **setGroupVisible**(`group`, `visible`): `void`

Gruppe per Code ein-/ausschalten.

#### Parameters

• **group**: `string`

• **visible**: `boolean`

#### Returns

`void`


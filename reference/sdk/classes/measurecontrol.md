# MeasureControl

mapx-gl MeasureControl

Strecken- und Flächenmessung per Klick (Kugelgeometrie, wie im Viewer).

## Extends

- `DrawControl`

## Constructors

### new MeasureControl()

> **new MeasureControl**(): [`MeasureControl`](/reference/sdk/classes/measurecontrol/)

#### Returns

[`MeasureControl`](/reference/sdk/classes/measurecontrol/)

#### Inherited from

`DrawControl.constructor`

## Methods

### onAdd()

> **onAdd**(`map`): `HTMLElement`

#### Parameters

• **map**: `Map$1`

#### Returns

`HTMLElement`

#### Overrides

`DrawControl.onAdd`

***

### onRemove()

> **onRemove**(): `void`

Unregister a control on the map and give it a chance to detach event listeners
and resources. This method is called by Map#removeControl
internally.

#### Returns

`void`

#### Inherited from

`DrawControl.onRemove`
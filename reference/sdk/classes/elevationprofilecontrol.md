# ElevationProfileControl

Linie klicken, Höhenprofil (Auf-/Abstieg, Diagramm) über /v1/elevation/profile.

## Extends

- `DrawControl`

## Constructors

### new ElevationProfileControl()

> **new ElevationProfileControl**(): [`ElevationProfileControl`](/reference/sdk/classes/elevationprofilecontrol/)

#### Returns

[`ElevationProfileControl`](/reference/sdk/classes/elevationprofilecontrol/)

#### Inherited from

`DrawControl.constructor`

## Properties

### last

> **last**: `null` \| [`ElevationProfile`](/reference/sdk/interfaces/elevationprofile/) = `null`

Letztes Profil (für eigene Auswertung).

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


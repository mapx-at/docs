# ElevationProfileControl

Source: https://docs.mapx.at/reference/sdk/classes/elevationprofilecontrol/

> mapx-gl ElevationProfileControl

Linie klicken, Höhenprofil (Auf-/Abstieg, Diagramm) über /v1/elevation/profile.

## Extends

- `DrawControl`

## Constructors

### new ElevationProfileControl()

> **new ElevationProfileControl**(): [`ElevationProfileControl`](https://docs.mapx.at/reference/sdk/classes/elevationprofilecontrol.md)

#### Returns

[`ElevationProfileControl`](https://docs.mapx.at/reference/sdk/classes/elevationprofilecontrol.md)

#### Inherited from

`DrawControl.constructor`

## Properties

### last

> **last**: `null` \| [`ElevationProfile`](https://docs.mapx.at/reference/sdk/interfaces/elevationprofile.md) = `null`

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

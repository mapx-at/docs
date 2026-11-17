# DeparturesPopup

Source: https://docs.mapx.at/reference/sdk/classes/departurespopup/

> mapx-gl DeparturesPopup

Klick auf die Karte (oder `open()`) zeigt die nächsten Abfahrten der Haltestelle in der Nähe.

## Constructors

### new DeparturesPopup()

> **new DeparturesPopup**(`map`, `o`): [`DeparturesPopup`](https://docs.mapx.at/reference/sdk/classes/departurespopup.md)

#### Parameters

• **map**: `Map$1`

• **o**: [`DeparturesPopupOptions`](https://docs.mapx.at/reference/sdk/interfaces/departurespopupoptions.md) = `{}`

#### Returns

[`DeparturesPopup`](https://docs.mapx.at/reference/sdk/classes/departurespopup.md)

## Methods

### open()

> **open**(`at`): `Promise`\<`void`\>

Popup für einen Punkt oder eine Haltestellen-ID öffnen.

#### Parameters

• **at**: `string` \| [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

#### Returns

`Promise`\<`void`\>

***

### remove()

> **remove**(): `void`

#### Returns

`void`

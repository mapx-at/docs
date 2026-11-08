# DeparturesPopup

Klick auf die Karte (oder `open()`) zeigt die nächsten Abfahrten der Haltestelle in der Nähe.

## Constructors

### new DeparturesPopup()

> **new DeparturesPopup**(`map`, `o`): [`DeparturesPopup`](/reference/sdk/classes/departurespopup/)

#### Parameters

• **map**: `Map$1`

• **o**: [`DeparturesPopupOptions`](/reference/sdk/interfaces/departurespopupoptions/) = `{}`

#### Returns

[`DeparturesPopup`](/reference/sdk/classes/departurespopup/)

## Methods

### open()

> **open**(`at`): `Promise`\<`void`\>

Popup für einen Punkt oder eine Haltestellen-ID öffnen.

#### Parameters

• **at**: `string` \| [`LL`](/reference/sdk/type-aliases/ll/)

#### Returns

`Promise`\<`void`\>

***

### remove()

> **remove**(): `void`

#### Returns

`void`


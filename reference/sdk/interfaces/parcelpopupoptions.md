# ParcelPopupOptions

mapx-gl ParcelPopupOptions

## Properties

### dossier?

> `optional` **dossier**: `boolean`

Voll-Dossier laden (`/v1/parcels/{id}`: Nutzung, Gebäude, Gelände, Adressen). Standard `true`.

***

### highlight?

> `optional` **highlight**: `boolean`

Parzellenumriss auf der Karte hervorheben. Standard `true`.

***

### onParcel()?

> `optional` **onParcel**: (`p`) => `void`

#### Parameters

• **p**: [`Dossier`](/reference/sdk/interfaces/dossier/) \| `Feature`\<`Geometry`, `GeoJsonProperties`\>

#### Returns

`void`
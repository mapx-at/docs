# ParcelPopupOptions

Source: https://docs.mapx.at/reference/sdk/interfaces/parcelpopupoptions/

> mapx-gl ParcelPopupOptions

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

• **p**: [`Dossier`](https://docs.mapx.at/reference/sdk/interfaces/dossier.md) \| `Feature`\<`Geometry`, `GeoJsonProperties`\>

#### Returns

`void`

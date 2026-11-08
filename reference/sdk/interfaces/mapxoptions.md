# MapxOptions

## Extends

- `Omit`\<`MapOptions`, `"style"` \| `"center"`\>

## Properties

### accessToken?

> `optional` **accessToken**: `string`

Öffentlicher Zugriffstoken (`mx_pk_…`, account.mapx.at). Gilt für alle weiteren Anfragen des SDK; alternativ `mapx.setAccessToken()`.

***

### center?

> `optional` **center**: [`LL`](/reference/sdk/type-aliases/ll/)

***

### style?

> `optional` **style**: `string` \| `string` & `object` \| `StyleSpecification`

Kurzname (`streets`, `satellite`, `outdoors`, ...), Style-URL oder Style-Objekt. Standard `streets`. Die alten deutschen Namen (`karte`, `luftbild`, ...) gelten bis 2027-05-31.

***

### terrain?

> `optional` **terrain**: `boolean` \| `object`

3D-Gelände einschalten (Überhöhung 1,4).


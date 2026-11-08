# setAccessToken

> **setAccessToken**(`token`): `void`

Zugriffstoken setzen (`mx_pk_…` im Browser, auf account.mapx.at erzeugt). Alle API-Anfragen senden ihn als `Authorization: Bearer`,
Karten (Style, Kacheln, Schriften, Sprites) als `access_token`-Query, Static-Image-URLs ebenfalls. Er geht nur an `mapx.at`-Hosts.

## Parameters

• **token**: `undefined` \| `null` \| `string`

## Returns

`void`


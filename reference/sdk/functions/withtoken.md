# withToken

mapx-gl withToken

> **withToken**(`u`, `token`): `string`

URL mit `access_token` versehen, wenn ein Token gesetzt ist, die URL zu mapx.at gehört und noch keinen Token trägt.
Bei `pmtiles://…/{z}/{x}/{y}` kommt er vor den Kachelpfad, bei Archiv-URLs ans Ende.

## Parameters

• **u**: `string`

• **token**: `string` = `API.token`

## Returns

`string`
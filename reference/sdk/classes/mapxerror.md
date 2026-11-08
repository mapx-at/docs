# MapxError

Fehler der mapx-API (`application/problem+json`, RFC 9457). Auf `code` verzweigen, nie auf den Text.

## Extends

- `Error`

## Constructors

### new MapxError()

> **new MapxError**(`init`): [`MapxError`](/reference/sdk/classes/mapxerror/)

#### Parameters

• **init**

• **init.allowed?**: `string`[]

• **init.code**: `string`

• **init.detail?**: `string`

• **init.param?**: `string`

• **init.retryAfter?**: `number`

• **init.status**: `number`

• **init.title?**: `string`

• **init.type?**: `string`

#### Returns

[`MapxError`](/reference/sdk/classes/mapxerror/)

#### Overrides

`Error.constructor`

## Properties

### allowed?

> `readonly` `optional` **allowed**: `string`[]

Erlaubte Werte bei Aufzählungen.

***

### code

> `readonly` **code**: `string`

Stabiler Code, z. B. `invalid-parameter`, `rate-limited`, `not-found`.

***

### detail

> `readonly` **detail**: `string`

Erklärung zu dieser Anfrage (englisch, mit `Accept-Language: de` deutsch).

***

### param?

> `readonly` `optional` **param**: `string`

Fehlerhafter Parameter (falls bekannt).

***

### retryAfter?

> `readonly` `optional` **retryAfter**: `number`

Wartezeit in Sekunden bei 429 (Header `Retry-After`).

***

### status

> `readonly` **status**: `number`

HTTP-Status (0 bei Netzwerkfehler).

***

### type?

> `readonly` `optional` **type**: `string`

URI des Fehlertyps (Anker auf docs.mapx.at/api/errors/).

## Methods

### fromResponse()

> `static` **fromResponse**(`r`): `Promise`\<[`MapxError`](/reference/sdk/classes/mapxerror/)\>

Fehler aus einer Antwort lesen (auch wenn der Body kein problem+json ist).

#### Parameters

• **r**: `Response`

#### Returns

`Promise`\<[`MapxError`](/reference/sdk/classes/mapxerror/)\>


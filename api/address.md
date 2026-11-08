# Address API

Address autofill for forms and address validation against the register.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Autofill completes a postal address while the user types. Validation checks an address against the official register.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](/guides/search/)

## Address Autofill: suggestions

`GET /v1/autofill/suggest`

Official address register only (BEV). Completes street, house number, postcode and place; fields separated in `suggestions[]`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Partial input, e.g. `Hauptpl 3 Mel` |
| `session_token` | query | string |  | – | Session id (1 to 64 characters), echoed back |
| `proximity` | query | string |  | – | lon,lat, prefers near addresses |
| `limit` | query | string |  | `5` | 1 to 10, default 5 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "session_token": "abc-1",
  "suggestions": [
    {
      "id": "address-1234567",
      "address_line1": "Hauptplatz 3",
      "street": "Hauptplatz",
      "housenumber": "3",
      "postcode": "3390",
      "place": "Melk",
      "full_address": "Hauptplatz 3, 3390 Melk",
      "coordinates": [
        15.3298,
        48.2273
      ]
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Address Autofill: retrieve the chosen address

`GET /v1/autofill/retrieve/{id}`

`id` from `suggestions[].id` (`address-<AdrCD>`). FeatureCollection with one point feature, properties as in suggest.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | Address id from suggest |
| `session_token` | query | string |  | – | As in suggest |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/autofill/retrieve/address-7065749" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/autofill/retrieve/address-7065749");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/autofill/retrieve/address-7065749")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          15.3298,
          48.2273
        ]
      },
      "properties": {
        "name": "Hauptplatz 3"
      }
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Address Validation (free text or structured)

`GET /v1/address/validate`

Checks an address against the official register. `status`: match | corrected | ambiguous | not_found. Returns the normalized address, `confidence` (0 to 1), `deviations[]` per field (kind: spelling | wrong | added), coordinate, `id` and `adrcd`. Understands Str./Straße, abbreviations, umlauts and ß.

<details>
<summary>More</summary>

`ambiguous` returns `candidates[]` (up to 5) instead of `address`.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string |  | – | Free text, e.g. `Hauptstr. 12a, 3390 Melk` (or structured fields) |
| `street` | query | string |  | – | Street |
| `housenumber` | query | string |  | – | House number |
| `postcode` | query | string |  | – | Postcode |
| `place` | query | string |  | – | Place |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/address/validate?q=Herrengasse%2013,%201010%20Wien" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/address/validate?q=Herrengasse%2013,%201010%20Wien");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/address/validate?q=Herrengasse%2013,%201010%20Wien")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "status": "corrected",
  "confidence": 0.93,
  "address": {
    "street": "Hauptstraße",
    "housenumber": "12a",
    "postcode": "3390",
    "place": "Melk"
  },
  "deviations": [
    {
      "field": "street",
      "kind": "spelling"
    }
  ],
  "id": "address-1234567",
  "coordinates": [
    15.33,
    48.23
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Address Validation (JSON body)

`POST /v1/address/validate`

Same as GET with the fields in a JSON object (`q` or `street`, `housenumber`, `postcode`, `place`). At most 16 KB.

**Request body (JSON)**

```json
{
  "street": "Hauptstr.",
  "housenumber": "12a",
  "postcode": "3390"
}
```

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -X POST "https://api.mapx.at/v1/address/validate" \
  -H "Content-Type: application/json" \
  -d '{"street":"Hauptstr.","housenumber":"12a","postcode":"3390"}' | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/address/validate", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    "street": "Hauptstr.",
    "housenumber": "12a",
    "postcode": "3390"
  }),
});
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.post("https://api.mapx.at/v1/address/validate", json={
    "street": "Hauptstr.",
    "housenumber": "12a",
    "postcode": "3390"
  })
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "status": "corrected",
  "confidence": 0.93,
  "address": {
    "street": "Hauptstraße",
    "housenumber": "12a",
    "postcode": "3390",
    "place": "Melk"
  },
  "deviations": [
    {
      "field": "street",
      "kind": "spelling"
    }
  ],
  "id": "address-1234567",
  "coordinates": [
    15.33,
    48.23
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

<Playground group="address" lang="en" />
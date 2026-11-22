# Address autofill and validation API

Source: https://docs.mapx.at/api/address/

> Address autofill for forms and address validation against the official Austrian address register: endpoints, parameters and examples.

Autofill completes a postal address while the user types. Validation checks an address against the official register.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](https://docs.mapx.at/guides/search.md)

## Address Autofill: suggestions

`GET /v1/autofill/suggest`

Official address register only (BEV). Completes street, house number, postcode and place; fields separated in `suggestions[]`. A complete place or postcode after the house number takes precedence as in geocode (`Hauptplatz 12a Graz` suggests addresses in Graz, nearest numbers first, if 12a does not exist there).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Partial input, e.g. `Hauptpl 3 Mel` |
| `session_token` | query | string |  | – | Session id (1 to 64 characters), echoed back |
| `proximity` | query | string |  | – | lon,lat, prefers near addresses |
| `limit` | query | string |  | `5` | 1 to 10, default 5 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Address Autofill: retrieve the chosen address

`GET /v1/autofill/retrieve/{id}`

`id` from `suggestions[].id` (`address-<AdrCD>`). FeatureCollection with one point feature; `features[].id` is the address id, `properties` carry the same fields as a suggestion (`id`, `address_line1`, `street`, `housenumber`, `postcode`, `place`, `municipality`, `state`, `full_address`, `coordinates`).

`session_token` echoes the parameter (`null` without it). 404 for an unknown id.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | Address id from suggest |
| `session_token` | query | string |  | – | As in suggest |

**Example**

**curl**

```bash
ID=$(curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=1" | jq -r '.suggestions[0].id')
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/autofill/retrieve/$ID" | jq
```

**JavaScript**

```js
const headers = { Authorization: "Bearer YOUR_TOKEN" };
const s = await (await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=1", { headers })).json();
const res = await fetch("https://api.mapx.at/v1/autofill/retrieve/" + encodeURIComponent(s.suggestions[0].id), { headers });
const data = await res.json();
```

**Python**

```python
import requests

h = {"Authorization": "Bearer YOUR_TOKEN"}
s = requests.get("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=1", headers=h).json()
res = requests.get("https://api.mapx.at/v1/autofill/retrieve/" + s["suggestions"][0]["id"], headers=h)
data = res.json()
```

**Example response**

```json
{
  "type": "FeatureCollection",
  "session_token": null,
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)",
  "features": [
    {
      "type": "Feature",
      "id": "address-7065749",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.242382,
          47.813929
        ]
      },
      "properties": {
        "id": "address-7065749",
        "address_line1": "Herrengasse 13",
        "street": "Herrengasse",
        "housenumber": "13",
        "postcode": "2700",
        "place": "Wiener Neustadt",
        "municipality": "Wiener Neustadt",
        "state": "Niederösterreich",
        "full_address": "Herrengasse 13, 2700 Wiener Neustadt",
        "coordinates": [
          16.242382,
          47.813929
        ]
      }
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `session_token` | string,null | The `session_token` parameter, echoed back (`null` without it). |
| `attribution` | string | Data sources (CC BY 4.0), attribution required. |
| `features` | array | One point feature; `id` is the address id `address-<AdrCD>`, `properties` hold `id`, `address_line1`, `street`, `housenumber`, `postcode`, `place`, `municipality`, `state`, `full_address` and `coordinates` [lon, lat]. |

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Address Validation (free text or structured)

`GET /v1/address/validate`

Checks an address against the official register. `status`: match | corrected | ambiguous | not_found. Returns the parsed `input`, the normalized `address`, `confidence` (0 to 1, 0 with `not_found`), `deviations[]` per field (kind: spelling | wrong | added | unknown), `coordinates`, `id` and `address_code`.

Understands Str./Straße, abbreviations, umlauts and ß; a house number range (`1-3`) is not the number 13. A given place or postcode takes precedence: candidates elsewhere only count if none lies there. A different house number is never a correction: if the number does not exist in the street, `status` is `not_found`, `deviations` holds `{"field": "housenumber", "kind": "unknown"}` and `candidates[]` lists the nearest numbers of that street (up to 5), e.g. `street=Rathausplatz&housenumber=999&postcode=3100`. With `ambiguous` the fields `address`, `coordinates`, `id` and `address_code` are `null` and `candidates[]` (up to 5, each shaped like `address`) lists the possible addresses, e.g. `q=Hauptplatz 3` (without place): `{"status": "ambiguous", "address": null, "id": null, "candidates": [{"id": "address-5111737", "full_address": "Hauptplatz 3, 6511 Zams", ...}, {"id": "address-6067644", "full_address": "Hauptplatz 3, 9170 Ferlach", ...}]}`. With `match` and `corrected`, `candidates` is empty.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string |  | – | Free text, e.g. `Haubtplatz 1, 3390 Melk` (or structured fields) |
| `street` | query | string |  | – | Street |
| `housenumber` | query | string |  | – | House number |
| `postcode` | query | string |  | – | Postcode |
| `place` | query | string |  | – | Place |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "status": "corrected",
  "confidence": 0.96,
  "input": {
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  },
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)",
  "deviations": [
    {
      "field": "street",
      "input": "Haubtplatz",
      "expected": "Hauptplatz",
      "kind": "spelling"
    }
  ],
  "address": {
    "id": "address-6444187",
    "address_line1": "Hauptplatz 1",
    "street": "Hauptplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk",
    "municipality": "Melk",
    "state": "Niederösterreich",
    "full_address": "Hauptplatz 1, 3390 Melk",
    "coordinates": [
      15.329515,
      48.227317
    ]
  },
  "coordinates": [
    15.329515,
    48.227317
  ],
  "id": "address-6444187",
  "address_code": "6444187",
  "candidates": []
}
```

| Field | Type | Description |
|---|---|---|
| `status` | string | `match`: found as given; `corrected`: found after corrections (see `deviations`); `ambiguous`: several addresses fit (see `candidates`); `not_found`: no address fits (if only the house number is unknown, `candidates` lists the nearest numbers of the street). |
| `confidence` | number | 0 to 1; 0 with `not_found`. |
| `input` | object | The input as understood: `street`, `housenumber`, `postcode`, `place` (parsed from `q` or taken from the structured fields). |
| `address` | object,null | The normalized address (`id`, `address_line1`, `street`, `housenumber`, `postcode`, `place`, `municipality`, `state`, `full_address`, `coordinates`); `null` unless `match` or `corrected`. |
| `deviations` | array | Per corrected field: `field`, `input`, `expected`, `kind` (spelling, wrong or added); with `not_found` for an unknown house number `{field: housenumber, kind: unknown}`. |
| `coordinates` | array,null | [lon, lat] of the address, `null` unless `match` or `corrected`. |
| `id` | string,null | Address id `address-<AdrCD>` (as in Autofill and geocoding). |
| `address_code` | string,null | Address code of the BEV address register. |
| `candidates` | array | With `ambiguous`: up to 5 possible addresses; with `not_found` for an unknown house number: up to 5 nearest numbers of the street; each shaped like `address`; otherwise empty. |
| `attribution` | string | Data sources (CC BY 4.0), attribution required. |

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Address Validation (JSON body)

`POST /v1/address/validate`

Same as GET with the fields in a JSON object (`q` or `street`, `housenumber`, `postcode`, `place`). At most 16 KB.

Needs a secret token (`mx_sk_`); public tokens get `403 pk-not-allowed`, so the playground cannot run it.

**Request body (JSON)**

```json
{
  "street": "Haubtplatz",
  "housenumber": "1",
  "postcode": "3390",
  "place": "Melk"
}
```

**Example**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/address/validate" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"street":"Haubtplatz","housenumber":"1","postcode":"3390","place":"Melk"}' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/address/validate", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify({
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  }),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/address/validate", headers={"Authorization": "Bearer YOUR_TOKEN"}, json={
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  })
data = res.json()
```

**Example response**

```json
{
  "status": "corrected",
  "confidence": 0.96,
  "input": {
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  },
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)",
  "deviations": [
    {
      "field": "street",
      "input": "Haubtplatz",
      "expected": "Hauptplatz",
      "kind": "spelling"
    }
  ],
  "address": {
    "id": "address-6444187",
    "address_line1": "Hauptplatz 1",
    "street": "Hauptplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk",
    "municipality": "Melk",
    "state": "Niederösterreich",
    "full_address": "Hauptplatz 1, 3390 Melk",
    "coordinates": [
      15.329515,
      48.227317
    ]
  },
  "coordinates": [
    15.329515,
    48.227317
  ],
  "id": "address-6444187",
  "address_code": "6444187",
  "candidates": []
}
```

| Field | Type | Description |
|---|---|---|
| `status` | string | `match`: found as given; `corrected`: found after corrections (see `deviations`); `ambiguous`: several addresses fit (see `candidates`); `not_found`: no address fits (if only the house number is unknown, `candidates` lists the nearest numbers of the street). |
| `confidence` | number | 0 to 1; 0 with `not_found`. |
| `input` | object | The input as understood: `street`, `housenumber`, `postcode`, `place` (parsed from `q` or taken from the structured fields). |
| `address` | object,null | The normalized address (`id`, `address_line1`, `street`, `housenumber`, `postcode`, `place`, `municipality`, `state`, `full_address`, `coordinates`); `null` unless `match` or `corrected`. |
| `deviations` | array | Per corrected field: `field`, `input`, `expected`, `kind` (spelling, wrong or added); with `not_found` for an unknown house number `{field: housenumber, kind: unknown}`. |
| `coordinates` | array,null | [lon, lat] of the address, `null` unless `match` or `corrected`. |
| `id` | string,null | Address id `address-<AdrCD>` (as in Autofill and geocoding). |
| `address_code` | string,null | Address code of the BEV address register. |
| `candidates` | array | With `ambiguous`: up to 5 possible addresses; with `not_found` for an unknown house number: up to 5 nearest numbers of the street; each shaped like `address`; otherwise empty. |
| `attribution` | string | Data sources (CC BY 4.0), attribution required. |

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

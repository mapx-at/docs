# Geocoding API

<!-- Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. -->

Turns an address or place name into coordinates and coordinates into the nearest address.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s), `heavy` (30/min) · Guide: [Search](/guides/search/)

## Forward geocoding

`GET /v1/geocode`

Search over the official address register, streets, places, municipalities, districts, postcodes and parcels. Road kilometers: `B83 km 329,7`, `A1 km 41` or `L190 km 3.5` (class A, S, B or L, number, the word `km`) return the point on the road at that kilometer as `place_type` `road_ref` with `properties.road_ref` (`ref`, `km`); the index holds a point every 0.1 km on motorways, expressways and federal roads and every 0.5 km on state roads (GIP.at), the nearest one is returned, the same state road number in several states gives several hits.

<details>
<summary>More</summary>

Response: FeatureCollection with `place_type`, `place_name`, `center`, `geometry`, `bbox`. Always pass `types` for cheap queries.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Search text (also path form `/v1/geocode/{q}.json`) |
| `limit` | query | integer (1–10) |  | `5` | 1 to 10, default 5 |
| `types` | query | string |  | – | Comma-separated result types: address, street, locality, place, district, region, postcode, poi, parcel (parcels only; input such as `KG 14105 Gst 102/3`, `14105/102/3`, `Landeck .102`), road_ref (road kilometer; input such as `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `proximity` | query | string |  | – | lon,lat; ranks equal matches by distance |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&limit=3&types=address" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&limit=3&types=address", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&limit=3&types=address", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "type": "FeatureCollection",
  "query": [
    "herrengasse",
    "13",
    "wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-1234567",
      "place_type": [
        "address"
      ],
      "place_name": "Herrengasse 13, 1010 Wien",
      "text": "Herrengasse 13",
      "center": [
        16.3665,
        48.2096
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3665,
          48.2096
        ]
      },
      "properties": {}
    }
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Forward geocoding, Mapbox path form

`GET /v1/geocode/{q}`

Same as `/v1/geocode?q=...`; the search text is the path segment (optionally with `.json`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | path | string | yes | – | Search text, URL-encoded |
| `limit` | query | string |  | `5` | 1 to 10, default 5 |
| `types` | query | string |  | – | Comma-separated result types: address, street, locality, place, district, region, postcode, poi, parcel (parcels only; input such as `KG 14105 Gst 102/3`, `14105/102/3`, `Landeck .102`), road_ref (road kilometer; input such as `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `proximity` | query | string |  | – | lon,lat |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode/Stephansplatz%20Wien.json?limit=3" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/geocode/Stephansplatz%20Wien.json?limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode/Stephansplatz%20Wien.json?limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "type": "FeatureCollection",
  "query": [
    "herrengasse",
    "13",
    "wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-1234567",
      "place_type": [
        "address"
      ],
      "place_name": "Herrengasse 13, 1010 Wien",
      "text": "Herrengasse 13",
      "center": [
        16.3665,
        48.2096
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3665,
          48.2096
        ]
      },
      "properties": {}
    }
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Reverse geocoding (nearest address, street, place)

`GET /v1/reverse`

Nearest features around a point, nearest first. Use `types` to restrict (e.g. `address`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lng` | query | string | yes | – | Longitude (also `lon`, `longitude`) |
| `lat` | query | string | yes | – | Latitude |
| `radius` | query | string |  | `500` | Meters, 1 to 5000, default 500 |
| `limit` | query | string |  | `1` | 1 to 10, default 1 |
| `types` | query | string |  | – | Like geocode |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&limit=1&types=address" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&limit=1&types=address", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&limit=1&types=address", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "type": "FeatureCollection",
  "query": [
    "herrengasse",
    "13",
    "wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-1234567",
      "place_type": [
        "address"
      ],
      "place_name": "Herrengasse 13, 1010 Wien",
      "text": "Herrengasse 13",
      "center": [
        16.3665,
        48.2096
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3665,
          48.2096
        ]
      },
      "properties": {}
    }
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Batch geocoding (up to 1000 requests)

`POST /v1/geocode/batch`

JSON array; each item has `q` or structured `street`/`housenumber`/`postcode`/`place`, optional `types` (string or list), `limit` (1 to 10), `proximity` [lon,lat], `bbox`. Response: array of FeatureCollections in input order. Requests run in batches via Meilisearch multi-search (at most 4 in parallel).

<details>
<summary>More</summary>

At most 512 KB.

</details>

**Request body (JSON)**

```json
[
  {
    "q": "Herrengasse 13 Wien",
    "types": "address",
    "limit": 1
  },
  {
    "street": "Hauptplatz",
    "housenumber": "3",
    "postcode": "3390"
  }
]
```

**Example**

**curl**


```bash
curl -s -X POST "https://api.mapx.at/v1/geocode/batch" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '[{"q":"Herrengasse 13 Wien","types":"address","limit":1},{"street":"Hauptplatz","housenumber":"3","postcode":"3390"}]' | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/geocode/batch", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify([
    {
      "q": "Herrengasse 13 Wien",
      "types": "address",
      "limit": 1
    },
    {
      "street": "Hauptplatz",
      "housenumber": "3",
      "postcode": "3390"
    }
  ]),
});
const data = await res.json();
```

**Python**


```python
import requests

res = requests.post("https://api.mapx.at/v1/geocode/batch", headers={"Authorization": "Bearer YOUR_TOKEN"}, json=[
    {
      "q": "Herrengasse 13 Wien",
      "types": "address",
      "limit": 1
    },
    {
      "street": "Hauptplatz",
      "housenumber": "3",
      "postcode": "3390"
    }
  ])
data = res.json()
```


**Example response**

```json
[
  {
    "type": "FeatureCollection",
    "query": [
      "herrengasse",
      "13",
      "wien"
    ],
    "features": [
      {
        "type": "Feature",
        "id": "address-1234567",
        "place_type": [
          "address"
        ],
        "place_name": "Herrengasse 13, 1010 Wien",
        "text": "Herrengasse 13",
        "center": [
          16.3665,
          48.2096
        ],
        "geometry": {
          "type": "Point",
          "coordinates": [
            16.3665,
            48.2096
          ]
        },
        "properties": {}
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `405` | Method not allowed |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground




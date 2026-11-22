# Geocoding API for Austria

Source: https://docs.mapx.at/api/geocoding/

> Geocoding API for Austria: forward, reverse and batch geocoding of addresses, streets, places, postcodes, with parameters and examples.

Turns an address or place name into coordinates and coordinates into the nearest address.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s), `heavy` (30/min) · Guide: [Search](https://docs.mapx.at/guides/search.md)

## Forward geocoding

`GET /v1/geocode`

Search over the official address register, streets, places, municipalities, districts, postcodes and points of interest. Road kilometers: `B83 km 329,7`, `A1 km 41` or `L190 km 3.5` (class A, S, B or L, number, the word `km`) return the point on the road at that kilometer as `place_type` `road_ref` with `properties.road_ref` (`ref`, `km`); the index holds a point every 0.1 km on motorways, expressways and federal roads and every 0.5 km on state roads (GIP.at), the nearest one is returned, the same state road number in several states gives several hits.

Addresses: when the input names a place or postcode after the house number (`Hauptplatz 12a Graz`, `Rathausplatz 1, 3100`), hits in that place come first; if the number does not exist there, the base number (`12` for `12a`), the street in that place and the nearest numbers follow, never the same number in another place first. House number ranges (`1-3`, `15 - 16`) stay ranges. Common abbreviations are expanded (`Hptpl.`, `Str.`, `Hauptstr.`, `Rathauspl.`, `Pl.`, `G.`, `Bhf`). Coordinates as search text (`16.37,48.21` or `48.21, 16.37`, both with a decimal point; order detected from Austria's range) answer like `/v1/reverse` with `query` [lon, lat]. Response: FeatureCollection with `place_type`, `place_name`, `center`, `geometry`, `bbox`; `relevance` is the match score of the search (0 to 1, lower for a fallback to the base number, street or a neighboring number); for addresses and streets `properties.address` holds the separate fields (`street`, `housenumber`, `postcode`, `place`, `locality`, `municipality`, `district`, `state`) and `properties.accuracy` is `rooftop` (point on the building), `parcel` (point on the plot) or `street`. Waters (lakes, rivers, streams from the BEV geographical names) are `place` hits with `properties.class` `see`, `fluss`, `bach` or `gewaesser`. `query` lists the search words without punctuation. Always pass `types` for cheap queries.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Search text (also path form `/v1/geocode/{q}.json`) |
| `limit` | query | integer (1–10) |  | `5` | 1 to 10, default 5 |
| `types` | query | string |  | – | Comma-separated result types: address, street, locality, place, district, region, postcode, poi, road_ref (road kilometer; input such as `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lon -180 to 180, lat -90 to 90; otherwise 400 `invalid-parameter`) |
| `proximity` | query | string |  | – | lon,lat; nearby matches (within about 40 km) rank ahead of distant ones that match only slightly better, matches beyond 40 km without special importance never ahead of nearby ones; exact place names stay on top; a search text naming a category (`Bahnhof`, `Museum`, see `/v1/searchbox/list/category`) also brings the nearest places of that category; ignored when bbox is set |

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

**Example response** (shortened: lists show only their first entries)

```json
{
  "type": "FeatureCollection",
  "query": [
    "Herrengasse",
    "13"
  ],
  "features": [
    {
      "id": "address-6792483",
      "type": "Feature",
      "place_type": [
        "address"
      ],
      "relevance": 0.97,
      "properties": {
        "mapx_type": "address",
        "class": null,
        "context": "Innere Stadt, 1010 Wien, W",
        "zoom": 18,
        "address": {
          "street": "Herrengasse",
          "housenumber": "13",
          "postcode": "1010",
          "place": "Wien",
          "locality": "Innere Stadt",
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "accuracy": "parcel"
      },
      "text": "Herrengasse 13",
      "place_name": "Herrengasse 13, Innere Stadt, 1010 Wien, W",
      "center": [
        16.365282,
        48.210154
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.365282,
          48.210154
        ]
      }
    }
  ],
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
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

## Forward geocoding, Mapbox path form

`GET /v1/geocode/{q}`

Same as `/v1/geocode?q=...`; the search text is the path segment (optionally with `.json`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | path | string | yes | – | Search text, URL-encoded |
| `limit` | query | string |  | `5` | 1 to 10, default 5 |
| `types` | query | string |  | – | Comma-separated result types: address, street, locality, place, district, region, postcode, poi, road_ref (road kilometer; input such as `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lon -180 to 180, lat -90 to 90; otherwise 400 `invalid-parameter`) |
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
    "Stephansplatz",
    "Wien"
  ],
  "features": [
    {
      "id": "station-at_49_1320",
      "type": "Feature",
      "place_type": [
        "poi"
      ],
      "relevance": 0.999,
      "properties": {
        "mapx_type": "station",
        "class": "station",
        "context": "Bahnhof, Wien",
        "zoom": 15,
        "category": "train_station"
      },
      "text": "Wien Stephansplatz",
      "place_name": "Wien Stephansplatz, Bahnhof, Wien",
      "center": [
        16.371607,
        48.208086
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.371607,
          48.208086
        ]
      }
    }
  ],
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
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

## Reverse geocoding (nearest address, street, place)

`GET /v1/reverse`

Nearest features around a point, nearest first, each with `properties.distance` (meters from the point) and the fields of geocode (`properties.address`, `properties.accuracy`). Use `types` to restrict (e.g. `address`). A coordinate that is not a number or out of range gives 400 `invalid-coordinates` with `param` `lng` or `lat`; `limit` and `radius` are checked separately (400 `invalid-parameter`).

For a point outside Austria the response has no features and a `warning` (often the order was swapped; it is lng, lat).

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
    16.3725,
    48.2085
  ],
  "features": [
    {
      "id": "address-6793782",
      "type": "Feature",
      "place_type": [
        "address"
      ],
      "relevance": 1,
      "properties": {
        "mapx_type": "address",
        "class": null,
        "context": "Innere Stadt, 1010 Wien, W",
        "zoom": 18,
        "address": {
          "street": "Stephansplatz",
          "housenumber": "1",
          "postcode": "1010",
          "place": "Wien",
          "locality": "Innere Stadt",
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "accuracy": "parcel",
        "distance": 27
      },
      "text": "Stephansplatz 1",
      "place_name": "Stephansplatz 1, Innere Stadt, 1010 Wien, W",
      "center": [
        16.372605,
        48.208728
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.372605,
          48.208728
        ]
      }
    }
  ],
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
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

## Batch geocoding (up to 1000 requests)

`POST /v1/geocode/batch`

JSON array; each item has `q` or structured `street`/`housenumber`/`postcode`/`place`, optional `types` (string or list), `limit` (1 to 10), `proximity` [lon,lat], `bbox`. Response: array in input order with one FeatureCollection per item (same fields as geocode, including the place precedence and coordinates as `q`); an invalid item does not fail the batch but is answered at its position with a problem object (RFC 9457: `type`, `title`, `status` 400, `detail`, `code`, `param`).

Needs a secret token (`mx_sk_`); public tokens get `403 pk-not-allowed`, so the playground cannot run it.

A malformed body (no array, empty, more than 1000 items) still gives 400 for the whole call. `proximity` only orders equally good matches here (no separate nearby search as in geocode). Items are searched in chunks, at most 4 in parallel; the whole batch counts as one request in the rate limit. At most 512 KB.

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

**Example response** (shortened: lists show only their first entries)

```json
[
  {
    "type": "FeatureCollection",
    "query": [
      "Herrengasse"
    ],
    "features": [
      {
        "id": "address-6792483",
        "type": "Feature",
        "place_type": [
          "address"
        ],
        "relevance": 0.97,
        "properties": {
          "mapx_type": "address",
          "class": null,
          "context": "Innere Stadt, 1010 Wien, W",
          "zoom": 18,
          "address": {
            "street": "Herrengasse",
            "housenumber": "13",
            "postcode": "1010",
            "place": "Wien",
            "locality": "Innere Stadt",
            "municipality": "Wien",
            "district": "Wien (Stadt)",
            "state": "Wien"
          },
          "accuracy": "parcel"
        },
        "text": "Herrengasse 13",
        "place_name": "Herrengasse 13, Innere Stadt, 1010 Wien, W",
        "center": [
          16.365282,
          48.210154
        ],
        "geometry": {
          "type": "Point",
          "coordinates": [
            16.365282,
            48.210154
          ]
        }
      }
    ],
    "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
  },
  {
    "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
    "title": "Invalid parameter",
    "status": 400,
    "detail": "types: unknown value 'parcel' (allowed: address, street, locality, place, district, region, postcode, poi, road_ref)",
    "code": "invalid-parameter",
    "param": "types",
    "allowed": [
      "address"
    ]
  }
]
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `405` | Method not allowed |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

# Search Box API: type-ahead search

Source: https://docs.mapx.at/api/search-box/

> Search Box API for autocomplete: suggestions while the user types, details of a result and places by category, with parameters and examples.

Type-ahead search for addresses and places. Call `suggest` while the user types, then `retrieve` for the chosen result.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](https://docs.mapx.at/guides/search.md)

## Search Box: suggestions

`GET /v1/searchbox/suggest`

Like geocode, but a suggestion list without coordinates (`mapbox_id` for `/retrieve`), with category (`poi_category`, `poi_category_ids`, `maki`) and `context`. Official POIs, no OSM data.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Search text |
| `session_token` | query | string |  | – | Arbitrary session id (up to 128 characters), no effect |
| `proximity` | query | string |  | – | lon,lat; same ranking as geocode, also returns the distance |
| `types` | query | string |  | – | region, postcode, district, place, locality, street, address, poi, road_ref (country, neighborhood, category have no effect) |
| `poi_category` | query | string |  | – | Comma-separated category ids (e.g. `gas_station`), see `/v1/searchbox/list/category` |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `limit` | query | string |  | `5` | 1 to 10, default 5 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "suggestions": [
    {
      "name": "Stephansdom",
      "mapbox_id": "poi-4711",
      "feature_type": "poi",
      "place_formatted": "Place of worship, Wien",
      "poi_category": [
        "place_of_worship"
      ]
    }
  ],
  "attribution": "mapx.at; ..."
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

## Search Box: resolve a suggestion

`GET /v1/searchbox/retrieve/{id}`

FeatureCollection with one feature (point, bbox, coordinates, context); 404 for an unknown id.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | `mapbox_id` from suggest |
| `session_token` | query | string |  | – | As in suggest |
| `proximity` | query | string |  | – | lon,lat, only for the distance |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/retrieve/place-6456531" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/retrieve/place-6456531", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/retrieve/place-6456531", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

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
          16.3731,
          48.2085
        ]
      },
      "properties": {
        "name": "Stephansdom"
      }
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
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Search Box: places of a category

`GET /v1/searchbox/category/{category}`

Places of the category (several comma-separated), nearest first with `proximity`. Ids as in `/v1/searchbox/list/category` (e.g. `gas_station`, `train_station`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `category` | path | string | yes | – | Category id(s) |
| `proximity` | query | string |  | – | lon,lat |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `limit` | query | string |  | `10` | 1 to 25, default 10 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

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
          16.37,
          48.21
        ]
      },
      "properties": {
        "name": "Tankstelle Am Stadtrand"
      }
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

## Search Box: category list

`GET /v1/searchbox/list/category`

Stable category ids (`canonical_id`), name and Maki icon.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `language` | query | string |  | – | de (default) or en |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/list/category?language=en", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/list/category?language=en", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "listItems": [
    {
      "canonical_id": "gas_station",
      "icon": "fuel",
      "name": "Tankstelle"
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

## Playground

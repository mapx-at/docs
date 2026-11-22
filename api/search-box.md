# Search Box API: type-ahead search

Source: https://docs.mapx.at/api/search-box/

> Search Box API for autocomplete: suggestions while the user types, details of a result and places by category, with parameters and examples.

Type-ahead search for addresses and places. Call `suggest` while the user types, then `retrieve` for the chosen result.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](https://docs.mapx.at/guides/search.md)

## Search Box: suggestions

`GET /v1/searchbox/suggest`

Search as you type over addresses, streets, places and official points of interest. FeatureCollection in the same format as geocode (schema `Place`, with point geometry); points of interest also carry `category` and `maki`, stops `stop_id` and `modes`. `/retrieve/{id}` returns a single hit by its `id`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Search text |
| `session_token` | query | string |  | – | Arbitrary session id (up to 128 characters), no effect |
| `proximity` | query | string |  | – | lng,lat; same ranking as geocode (including the nearest places of a category named in `q`), also returns the distance |
| `types` | query | string |  | – | region, postcode, district, place, locality, street, address, poi, road_ref (country, neighborhood, category have no effect) |
| `poi_category` | query | string |  | – | Comma-separated category ids (e.g. `fuel`), see `/v1/searchbox/list/category` |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lng -180 to 180, lat -90 to 90; otherwise 400 `invalid-parameter`) |
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
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "place-7158611",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3731,
          48.2085
        ]
      },
      "properties": {
        "type": "place",
        "name": "Stephansdom",
        "place_formatted": "Kirche und Kapelle, Wien",
        "relevance": 0.99,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "zoom": 16,
        "class": "kirche",
        "category": "place_of_worship_church",
        "maki": "religious-christian"
      }
    }
  ],
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
}
```

| Field | Type | Description |
|---|---|---|
| `query` | array | Geocoding only: the search words, or [lng, lat] for reverse geocoding and coordinates as search text. |
| `features` | array | Hits, best or nearest first. Each is a GeoJSON point feature with a stable `id` (and `bbox` for areas); `properties` always hold `type`, `name`, `place_formatted`, `relevance`, `accuracy` and `address`, depending on the hit also `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (schema `Place`). |
| `warning` | string | Reverse geocoding only: the point is outside Austria. |
| `attribution` | string | Data sources (CC BY 4.0), attribution required. |

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

FeatureCollection with the one hit, in the same format as suggest and geocode (schema `Place`); 404 for an unknown id.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `id` | path | string | yes | – | `id` of a hit from suggest, geocode or category |
| `session_token` | query | string |  | – | As in suggest |
| `proximity` | query | string |  | – | lng,lat, only for the distance |

**Example**

**curl**

```bash
ID=$(curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&limit=1" | jq -r '.features[0].id')
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/retrieve/$ID" | jq
```

**JavaScript**

```js
const headers = { Authorization: "Bearer YOUR_TOKEN" };
const s = await (await fetch("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&limit=1", { headers })).json();
const res = await fetch("https://api.mapx.at/v1/searchbox/retrieve/" + encodeURIComponent(s.features[0].id), { headers });
const data = await res.json();
```

**Python**

```python
import requests

h = {"Authorization": "Bearer YOUR_TOKEN"}
s = requests.get("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&limit=1", headers=h).json()
res = requests.get("https://api.mapx.at/v1/searchbox/retrieve/" + s["features"][0]["id"], headers=h)
data = res.json()
```

**Example response**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "place-7158611",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3731,
          48.2085
        ]
      },
      "properties": {
        "type": "place",
        "name": "Stephansdom",
        "place_formatted": "Kirche und Kapelle, Wien",
        "relevance": 0.99,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "zoom": 16,
        "class": "kirche",
        "category": "place_of_worship_church",
        "maki": "religious-christian"
      }
    }
  ],
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
}
```

| Field | Type | Description |
|---|---|---|
| `query` | array | Geocoding only: the search words, or [lng, lat] for reverse geocoding and coordinates as search text. |
| `features` | array | Hits, best or nearest first. Each is a GeoJSON point feature with a stable `id` (and `bbox` for areas); `properties` always hold `type`, `name`, `place_formatted`, `relevance`, `accuracy` and `address`, depending on the hit also `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (schema `Place`). |
| `warning` | string | Reverse geocoding only: the point is outside Austria. |
| `attribution` | string | Data sources (CC BY 4.0), attribution required. |

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

Places of the category (several comma-separated), nearest first with `proximity` (then with `properties.distance`). Ids as in `/v1/searchbox/list/category` (e.g. `fuel`, `railway_station`). Same format as suggest (schema `Place`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `category` | path | string | yes | – | Category id(s) |
| `proximity` | query | string |  | – | lng,lat |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lng -180 to 180, lat -90 to 90; otherwise 400 `invalid-parameter`) |
| `limit` | query | string |  | `10` | 1 to 25, default 10 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/category/fuel?proximity=16.3725,48.2085&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/category/fuel?proximity=16.3725,48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/category/fuel?proximity=16.3725,48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "poi-51234",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.37,
          48.21
        ]
      },
      "properties": {
        "type": "poi",
        "name": "Tankstelle Am Stadtrand",
        "place_formatted": "Tankstelle, Wien",
        "relevance": 1,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "distance": 412,
        "zoom": 16,
        "category": "fuel",
        "maki": "fuel"
      }
    }
  ],
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
}
```

| Field | Type | Description |
|---|---|---|
| `query` | array | Geocoding only: the search words, or [lng, lat] for reverse geocoding and coordinates as search text. |
| `features` | array | Hits, best or nearest first. Each is a GeoJSON point feature with a stable `id` (and `bbox` for areas); `properties` always hold `type`, `name`, `place_formatted`, `relevance`, `accuracy` and `address`, depending on the hit also `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (schema `Place`). |
| `warning` | string | Reverse geocoding only: the point is outside Austria. |
| `attribution` | string | Data sources (CC BY 4.0), attribution required. |

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
      "canonical_id": "fuel",
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

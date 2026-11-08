# Search Box API

Suggestions while typing, details of a result and places by category.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Type-ahead search for addresses and places. Call `suggest` while the user types, then `retrieve` for the chosen result.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](/guides/search/)

## Search Box: suggestions

`GET /v1/searchbox/suggest`

Like geocode, but a suggestion list without coordinates (`mapbox_id` for `/retrieve`), with category (`poi_category`, `poi_category_ids`, `maki`) and `context`. Official POIs, no OSM data.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string | yes | – | Search text |
| `session_token` | query | string |  | – | Arbitrary session id (up to 128 characters), no effect |
| `proximity` | query | string |  | – | lon,lat |
| `types` | query | string |  | – | region, postcode, district, place, locality, street, address, poi, parcel, road_ref (country, neighborhood, category have no effect) |
| `poi_category` | query | string |  | – | Comma-separated category ids (English, e.g. `pharmacy`; old German ids deprecated until 2027-05-31), see `/v1/searchbox/list/category` |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `limit` | query | string |  | `5` | 1 to 10, default 5 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3")
data = res.json()
```

</TabItem>
</Tabs>

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
  "attribution": "..."
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
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

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/searchbox/retrieve/place-6456531" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/retrieve/place-6456531");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/searchbox/retrieve/place-6456531")
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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Search Box: places of a category

`GET /v1/searchbox/category/{category}`

Places of the category (several comma-separated), nearest first with `proximity`. Ids are English (e.g. `pharmacy`, `train_station`); the old German ids (`apotheke`, `bahnhof`, ...) work as aliases until 2027-05-31 (`Deprecation`, `Sunset`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `category` | path | string | yes | – | Category id(s) |
| `proximity` | query | string |  | – | lon,lat |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `limit` | query | string |  | `10` | 1 to 25, default 10 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=16.3725,48.2085&limit=3" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=16.3725,48.2085&limit=3");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=16.3725,48.2085&limit=3")
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
          16.37,
          48.21
        ]
      },
      "properties": {
        "name": "Apotheke Zum Hirschen"
      }
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

## Search Box: category list

`GET /v1/searchbox/list/category`

Stable category ids (`canonical_id`), name and Maki icon.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `language` | query | string |  | – | de (default) or en |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/list/category?language=en");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/searchbox/list/category?language=en")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "listItems": [
    {
      "canonical_id": "pharmacy",
      "icon": "pharmacy",
      "name": "Apotheke"
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

<Playground group="search-box" lang="en" />
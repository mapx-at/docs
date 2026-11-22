# Tilequery API

Source: https://docs.mapx.at/api/tilequery/

> Tilequery API: features around a point from any vector tileset, with parameters and examples.

Tilequery returns the vector features near a coordinate.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Maps](https://docs.mapx.at/guides/maps.md)

## Tilequery: vector features around a point

`GET /v1/tilequery/{tilesets}/{coordinate}.json`

Reads the vector tile at the tileset's highest zoom (and neighbouring tiles for a radius). FeatureCollection, nearest feature first, `properties.tilequery = {distance (m), geometry (point|linestring|polygon), layer}`. Polygons containing the point have distance 0.

Several comma-separated tilesets add `tilequery.tileset`; `dtm`/`dsm` return the height as a point feature. Geometries are clipped to the tile.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `tilesets` | path | string | yes | – | Tileset names, comma-separated (at most 15), e.g. `streets,contours` |
| `coordinate` | path | string | yes | – | lng,lat (the path ends in `.json`) |
| `radius` | query | string |  | `0` | Meters, 0 to 2000, default 0 |
| `limit` | query | string |  | `5` | 1 to 50, default 5 |
| `layers` | query | string |  | – | Comma-separated layer names |
| `geometry` | query | string |  | – | point, linestring or polygon |
| `dedupe` | query | string |  | – | true (default) removes duplicates across tile borders |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tilequery/streets/16.3725,48.2085.json?radius=50&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/tilequery/streets/16.3725,48.2085.json?radius=50&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/tilequery/streets/16.3725,48.2085.json?radius=50&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
        "type": "LineString",
        "coordinates": [
          [
            16.3721649,
            48.2086397
          ],
          [
            16.3722213,
            48.208612
          ]
        ]
      },
      "properties": {
        "kind": "path",
        "kind_detail": "path",
        "min_zoom": 14,
        "sort_rank": 11,
        "structure": "tunnel",
        "tilequery": {
          "distance": 24.2,
          "geometry": "linestring",
          "layer": "road_l"
        }
      }
    }
  ]
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

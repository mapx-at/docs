# Speed limits API: statutory limits from GIP.at

Source: https://docs.mapx.at/api/speed-limits/

> Speed limits API for Austria: statutory speed limits of the road network in a bounding box or along a route corridor, with parameters and examples.

Statutory speed limits of road sections as GeoJSON. Limits shown on motorway gantries: [Traffic API](https://docs.mapx.at/api/traffic.md).

Base URL: `https://api.mapx.at` · Rate-limit zone: `route` (5/s) · Guide: [Navigation](https://docs.mapx.at/guides/navigation.md)

## Speed limits (GIP.at, statutory limits only)

`GET /v1/speedlimits`

Statutory speed limits of the road network as compact GeoJSON, either inside a box (`bbox`, at most 0.08 square degrees, about 28 x 28 km) or in a corridor around a route (`corridor` as polyline6, e.g. `routes[].geometry` with `geometries=polyline6`, plus `buffer` in meters).

One LineString feature per road link in digitizing direction with `limit_kmh`, `direction` (`both`, `forward` or `backward` relative to the geometry) and `id` (GIP link id); if the directions differ there is one feature per direction. Only statutory limits (5, 10, 20 ... 130 km/h, including the default limits inside built-up areas and on rural roads) are returned. Links without a value (many motorways) and modeled speeds such as 53 or 84 km/h are left out. At most 60,000 features. Source: GIP.at (CC BY 4.0).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat in degrees, at most 0.08 square degrees; instead of corridor |
| `corridor` | query | string |  | – | Route as polyline6 (at most 20,000 points); instead of bbox |
| `buffer` | query | string |  | `30` | With corridor only: meters around the route, 1 to 500, default 30 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "type": "FeatureCollection",
  "legal_only": true,
  "attribution": "mapx.at; GIP.at (CC BY 4.0)",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            15.3294,
            48.2272
          ],
          [
            15.3301,
            48.2275
          ]
        ]
      },
      "properties": {
        "limit_kmh": 50,
        "direction": "both",
        "id": 1543486
      }
    }
  ]
}
```

| Field | Type | Description |
|---|---|---|
| `legal_only` | boolean | always true: only statutory limits are listed |

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

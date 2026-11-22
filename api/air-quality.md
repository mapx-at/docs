# Air quality API for Austria

Source: https://docs.mapx.at/api/air-quality/

> Air quality index, pollutants and Saharan dust for a point in Austria from GeoSphere Austria: endpoints, parameters and example requests.

Daily air quality index and hourly nitrogen dioxide, ozone, PM10 and PM2.5 near the surface plus the Saharan dust load for a point, from the GeoSphere chemical weather model (3 km). The index follows the EEA European Air Quality Index (1 good to 6 extremely poor; SO2 is not provided by the model). Map layers: the weather tiles `aqi`, `no2`, `o3`, `pm10`, `pm25` and `dust` (see [Weather API](https://docs.mapx.at/api/weather.md)). Source: GeoSphere Austria, CC BY 4.0.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Maps](https://docs.mapx.at/guides/maps.md)

## Air quality index and pollutants for a point

`GET /v1/air-quality/{coordinate}`

GeoSphere chemical weather model (3 km) bilinear to the point: daily index `aqi` (3 days) and hourly NO2, O3, PM10, PM2.5 near the surface in ug/m3 with an hourly index (worst level by the EEA European Air Quality Index limits; SO2 is not provided by the model) plus the Saharan dust load (mg/m2, 0.2 degree grid).

Austria and surroundings. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinate` | path | string | yes | – | lon,lat, e.g. 16.37,48.21 |
| `hours` | query | integer |  | `24` | 1 to 72, default 24; the daily run (00 UTC) reaches 48 h, fewer hours are marked with `truncated: true` |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/air-quality/16.3725,48.2085?hours=6" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/air-quality/16.3725,48.2085?hours=6", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/air-quality/16.3725,48.2085?hours=6", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "location": {
    "lon": 16.37,
    "lat": 48.21
  },
  "model": "GeoSphere chemical weather forecast",
  "reference_time": "2026-11-08T00:00:00Z",
  "index_scale": {
    "1": "good",
    "2": "fair",
    "3": "moderate",
    "4": "poor",
    "5": "very_poor",
    "6": "extremely_poor"
  },
  "daily": [
    {
      "date": "2026-11-08",
      "index": 2,
      "category": "fair"
    }
  ],
  "hourly": [
    {
      "time": "2026-11-08T12:00:00Z",
      "index": 2,
      "category": "fair",
      "no2_ugm3": 18.2,
      "o3_ugm3": 45.1,
      "pm10_ugm3": 21,
      "pm25_ugm3": 9.4,
      "saharan_dust_load_mgm2": 0
    }
  ],
  "attribution": "mapx.at; Luftqualität: GeoSphere Austria, CC BY 4.0 (https://data.hub.geosphere.at)"
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
| `503` | Data temporarily unavailable |

## Playground

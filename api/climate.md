# Climate API: solar, snow, heat days

Source: https://docs.mapx.at/api/climate/

> Climate API for a point in Austria: solar potential, climate normals, heat days, snow and drought from GeoSphere Austria, with parameters and examples.

Climate of a point: yearly solar radiation and sunshine hours, temperature and precipitation normals with trend, hot and frost days, snow days per winter and the drought index. Source: GeoSphere Austria, CC BY 4.0.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Maps](https://docs.mapx.at/guides/maps.md)

## Climate at a point (solar potential, normals, heat days, snow, drought)

`GET /v1/climate/{coordinate}`

Climate of one point from open data of GeoSphere Austria: solar potential (yearly global radiation in kWh/m2 and modelled sunshine hours, mean of the last 3 complete years, 100 m), temperature and precipitation (normal 1991-2020, last 10 years, trend, monthly normals, 1 km), hot/summer/frost/ice days and tropical nights per year, snow days per winter and the current snow depth, and the drought index SPEI (90 and 365 days, latest value with class).

Each section has `source` (dataset, cell, last day of data) and `retrieved_at`. Values are derived per grid cell and cached; the first request for a new cell fetches them from the source and can take 10 to 20 seconds (`sections` limits the work), later requests come from the cache. Outside Austria 404 `outside-coverage`; unreadable coordinates 400 `invalid-coordinates`; 503 while the source is unavailable or rate limited. Data source: GeoSphere Austria (APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE), CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinate` | path | string | yes | – | lon,lat or lon,lat.json |
| `sections` | query | string |  | `all` | Comma-separated sections to return (default: all) |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "location": [
    15.3294,
    48.2272
  ],
  "status": "complete",
  "solar": {
    "period": "2023-2025",
    "years": 3,
    "global_radiation_kwh_m2_year": 1287.1,
    "global_radiation_horizontal_kwh_m2_year": 1219.1,
    "direct_radiation_kwh_m2_year": 911.6,
    "diffuse_radiation_kwh_m2_year": 375.5,
    "model_sunshine_hours_year": 2949.5,
    "model_relative_sunshine_pct": 60.3,
    "by_year": [
      {
        "year": 2025,
        "global_radiation_kwh_m2": 1310.8,
        "model_sunshine_hours": 2958.4
      }
    ],
    "source": {
      "provider": "GeoSphere Austria",
      "dataset": "apolis-v2-1d-100m",
      "name": "APOLIS solar radiation",
      "license": "CC BY 4.0",
      "resolution": 100,
      "cell_center": [
        15.33149,
        48.22776
      ],
      "data_until": "2025-12-31"
    },
    "retrieved_at": "2026-11-08T07:31:55Z"
  },
  "sunshine": {
    "hours_year": {
      "normal": 1890.2,
      "normal_period": "1991-2020",
      "recent": 2041.7,
      "recent_period": "2016-2025"
    }
  },
  "snow": {
    "snow_days_per_winter": {
      "normal": 36.6,
      "recent": 18.4
    },
    "max_depth_cm": {
      "normal": 8.3,
      "recent": 6.1
    },
    "current": {
      "depth_cm": 0,
      "date": "2026-11-06"
    }
  },
  "attribution": "mapx.at; GeoSphere Austria APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE (CC BY 4.0)"
}
```

| Field | Type | Description |
|---|---|---|
| `location` | array | [longitude, latitude] (WGS84), optional height. |
| `status` | string | `partial`: some sections are still being loaded |
| `solar` | object,null | Solar potential: yearly sums, mean of the last 3 complete years (APOLIS, 100 m) |
| `temperature` | object,null | Air temperature (SPARTACUS, 1 km) |
| `precipitation` | object,null | Precipitation (SPARTACUS, 1 km) |
| `sunshine` | object,null | Measured sunshine duration (station-based SPARTACUS grid, 1 km); differs from the modelled APOLIS value in `solar` |
| `temperature_extremes` | object,null | Hot, summer, frost and ice days per year, tropical nights (SPARTACUS daily data, 1 km); normal period 1991-2020 and last 10 complete years |
| `snow` | object,null | Snow cover (SNOWGRID-CL model, 1 km) |
| `drought` | object,null | Drought index SPEI (WINFORE, 1 km), latest value |

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

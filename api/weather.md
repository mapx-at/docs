# Weather API

<!-- Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. -->

Observations of the nearest stations, an hourly forecast for a point and raster tiles for a map overlay. Source: GeoSphere Austria, CC BY 4.0.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Maps](/guides/maps/)

## Current observations of the nearest TAWES stations

`GET /v1/weather/current`

GeoSphere Austria station network, 10-minute values (temperature, humidity, wind, gust, precipitation, pressure). Nearest station first. Values come from a local cache, never fetched per request. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lon` | query | string | yes | – | Longitude (also `lng`), WGS84 |
| `lat` | query | string | yes | – | Latitude |
| `limit` | query | string |  | `3` | 1 to 10, default 3 |
| `radius` | query | string |  | `50` | Search radius in km, default 50 |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/current?lon=16.3725&lat=48.2085&limit=3" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/weather/current?lon=16.3725&lat=48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/current?lon=16.3725&lat=48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
          16.35,
          48.25
        ]
      },
      "properties": {
        "station_id": "11010",
        "name": "Wien Hohe Warte",
        "state": "Wien",
        "altitude_m": 198,
        "distance_m": 4200,
        "time": "2026-11-08T11:50:00Z",
        "temperature_c": 9.4,
        "humidity_pct": 71,
        "wind_speed_ms": 3.1,
        "wind_direction_deg": 290,
        "pressure_hpa": 1002.1
      }
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Hourly point forecast (AROME, 1 km)

`GET /v1/weather/forecast`

GeoSphere Austria model nwp-v2-1h-1km, bilinear to the point, from the current hour. Austria plus margin. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lon` | query | string | yes | – | Longitude (also `lng`), WGS84 |
| `lat` | query | string | yes | – | Latitude |
| `hours` | query | string |  | `24` | 1 to 60, default 24 |
| `ensemble` | query | string |  | – | true: add `uncertainty` (p10/p50/p90 of the ensemble for temperature, gust, precipitation) to every hour |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/forecast?lon=16.3725&lat=48.2085&hours=6&ensemble=true" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/weather/forecast?lon=16.3725&lat=48.2085&hours=6&ensemble=true", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/forecast?lon=16.3725&lat=48.2085&hours=6&ensemble=true", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "location": [
    16.37,
    48.21
  ],
  "reference_time": "2026-11-08T06:00:00Z",
  "hours": [
    {
      "time": "2026-11-08T12:00:00Z",
      "temperature_c": 10.1,
      "precipitation_mm": 0,
      "wind_speed_ms": 3.4,
      "wind_direction_deg": 285,
      "humidity_pct": 68,
      "cloud_cover_pct": 40,
      "pressure_msl_hpa": 1008.3,
      "weather_symbol": {
        "code": 3,
        "text": "Partly cloudy",
        "icon": "partly-cloudy"
      },
      "snow_limit_m": 1200,
      "sunshine_min": 31.5,
      "radiation_wm2": 180.2,
      "cape_jkg": 0,
      "snowfall_mm": 0,
      "uncertainty": {
        "temperature_c": {
          "p10": 8.9,
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Precipitation nowcast for a point (15 min steps, 3 h)

`GET /v1/weather/nowcast/{coordinates}`

GeoSphere INCA nowcast nowcast-v1-15min-1km, bilinear to the point. `precipitation_starts_in_min` / `precipitation_ends_in_min` are derived from the 15-minute steps (wet from 0.1 mm/h); `null` when nothing changes within the horizon. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinates` | path | string | yes | – | lon,lat, e.g. 16.3725,48.2085 |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/nowcast/16.3725,48.2085" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/weather/nowcast/16.3725,48.2085", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/nowcast/16.3725,48.2085", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "location": {
    "lon": 16.3725,
    "lat": 48.2085
  },
  "reference_time": "2026-11-08T10:45Z",
  "interval": "15min",
  "precipitation_now": false,
  "precipitation_starts_in_min": 30,
  "precipitation_ends_in_min": null,
  "steps": [
    {
      "time": "2026-11-08T11:00Z",
      "minutes_from_now": 5,
      "precipitation_mm": 0,
      "precipitation_rate_mm_h": 0,
      "precipitation_type": null,
      "temperature_c": 9.8,
      "dew_point_c": 5.1,
      "humidity_pct": 71,
      "wind_speed_ms": 3.2,
      "wind_gust_ms": 6.1,
      "wind_direction_deg": 290
    }
  ]
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## TileJSON of weather tiles with time dimension

`GET /v1/weather/tiles/{param}.json`

param: temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall (hourly, AROME) or rr_nowcast, pt_nowcast (precipitation radar nowcast, 15 min steps). `mapx:time.times` lists the available hours (UTC), `mapx:time.template` has the placeholder `{time}`; `tiles` uses `now`.

<details>
<summary>More</summary>

Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `param` | path | string | yes | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast or pt_nowcast, with .json |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/tiles/temperature.json" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/weather/tiles/temperature.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/tiles/temperature.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "tilejson": "3.0.0",
  "tiles": [
    "https://api.mapx.at/v1/weather/tiles/temperature/now/{z}/{x}/{y}.png"
  ],
  "minzoom": 0,
  "maxzoom": 14
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Weather raster tile (PNG 256, transparent outside the data)

`GET /v1/weather/tiles/{param}/{time}/{z}/{x}/{y}.png`

Color ramp per parameter (see legend in the TileJSON). time: `now` or a UTC hour such as `2026-11-07T20:00Z`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `param` | path | string | yes | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast, pt_nowcast |
| `time` | path | string | yes | – | now or 2026-11-07T20:00Z (nowcast: 15 min steps such as 2026-11-07T20:15Z) |
| `z` | path | integer | yes | – | 0 to 14 |
| `x` | path | integer | yes | – | Column |
| `y` | path | string | yes | – | Row, with .png |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/tiles/temperature/now/6/34/22.png" -o result.png
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/weather/tiles/temperature/now/6/34/22.png", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/tiles/temperature/now/6/34/22.png", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.png", "wb").write(res.content)
```


**Example response**: `image/png` · PNG; header X-Weather-Reference-Time

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Official weather warnings (GeoSphere Austria)

`GET /v1/weather/warnings`

Active and upcoming official weather warnings (up to 4 days ahead) as GeoJSON MultiPolygons in WGS84, one feature per warning (merged municipalities). `type`: wind, rain, snow, black_ice, thunderstorm, heat, cold; `level`: 1 yellow, 2 orange, 3 red; `text`, `impact` and `recommendation` carry `de` and `en`.

<details>
<summary>More</summary>

Fetched centrally every 5 minutes, never per request. Without `bbox` and `lon`/`lat` all warnings are returned. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lon` | query | string |  | – | Longitude: only warnings that contain the point (with lat) |
| `lat` | query | string |  | – | Latitude |
| `bbox` | query | string |  | – | west,south,east,north in degrees (WGS84): only warnings touching the box |
| `type` | query | string |  | – | wind, rain, snow, black_ice, thunderstorm, heat or cold |
| `min_level` | query | string |  | – | 1 (yellow, default) to 3 (red) |
| `active` | query | string |  | – | now: only warnings valid right now; all (default): also upcoming |

**Example**

**curl**


```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/warnings?bbox=9.4,46.3,17.3,49.1&min_level=1" | jq
```

**JavaScript**


```js
const res = await fetch("https://api.mapx.at/v1/weather/warnings?bbox=9.4,46.3,17.3,49.1&min_level=1", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**


```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/warnings?bbox=9.4,46.3,17.3,49.1&min_level=1", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```


**Example response**

```json
{
  "type": "FeatureCollection",
  "updated": "2026-11-08T08:00Z",
  "attribution": "Warnungen: GeoSphere Austria, CC BY 4.0 (https://warnungen.zamg.at)",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
          [
            [
              [
                11.39,
                47.26
              ],
              [
                11.41,
                47.26
              ],
              [
                11.41,
                47.28
              ],
              [
                11.39,
                47.26
              ]
            ]
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Playground




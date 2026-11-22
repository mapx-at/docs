# Weather API: forecast and nowcast

Source: https://docs.mapx.at/api/weather/

> Weather API for Austria: current observations, hourly forecast, precipitation nowcast, warnings and weather tiles from GeoSphere Austria.

Observations of the nearest stations, an hourly forecast for a point and raster tiles for a map overlay. Source: GeoSphere Austria, CC BY 4.0.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Maps](https://docs.mapx.at/guides/maps.md)

## Current observations of the nearest TAWES stations

`GET /v1/weather/current`

GeoSphere Austria station network, 10-minute values (temperature, humidity, wind, gust, precipitation, pressure). Nearest station first; compare `altitude_m` of the station with `location.elevation_m` of the point (terrain height, mapx 1 m DTM). Values come from a local cache, never fetched per request.

Points outside Austria and surroundings: 404 `outside-coverage`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lon` | query | string | yes | – | Longitude (also `lng`), WGS84 |
| `lat` | query | string | yes | – | Latitude |
| `limit` | query | string |  | `3` | 1 to 10, default 3 |
| `radius` | query | string |  | `50000` | Search radius in metres, 1 to 600000, default 50000 |

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
  "location": {
    "lon": 16.3725,
    "lat": 48.2085,
    "elevation_m": 171.4
  },
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.35667,
          48.24861
        ]
      },
      "properties": {
        "station_id": "11035",
        "name": "Wien/Hohe Warte",
        "state": "Wien",
        "altitude_m": 198,
        "distance_m": 4600,
        "time": "2026-11-08T11:50:00Z",
        "temperature_c": 9.4,
        "humidity_pct": 71,
        "wind_speed_ms": 3.1,
        "wind_gust_ms": 5.8,
        "wind_direction_deg": 290,
        "precipitation_mm_10min": 0,
        "pressure_hpa": 1002.1,
        "pressure_msl_hpa": 1018.6
      }
    }
  ],
  "time": "2026-11-08T11:50:00Z",
  "attribution": "mapx.at; Datenquelle: GeoSphere Austria, CC BY 4.0 (https://data.hub.geosphere.at)"
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

## Hourly point forecast (AROME, 1 km)

`GET /v1/weather/forecast`

GeoSphere Austria model nwp-v2-1h-1km (runs every 3 h), bilinear to the point, from the current hour. Austria plus margin; outside 404 `outside-coverage`. A new run is published with its first 24 hours and extended in the background; until then the later hours come from the previous run (`previous_run`).

`weather_symbol.code` is the GeoSphere symbol 1 to 32: 1 clear, 2 mostly clear, 3 partly cloudy, 4 mostly cloudy, 5 overcast, 6-7 fog, 8-13 rain, 14-19 snow, 20-25 sleet, 26-32 thunderstorm (no day/night distinction). Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lon` | query | string | yes | – | Longitude (also `lng`), WGS84 |
| `lat` | query | string | yes | – | Latitude |
| `hours` | query | string |  | `24` | 1 to 48, default 24; larger values give 400 |
| `ensemble` | query | string |  | – | true or false (default). true: add `uncertainty` (p10/p50/p90 of the ensemble for temperature, gust, precipitation) to the hours the ensemble covers (the next 24 h); see `ensemble.status` |

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
  "location": {
    "lon": 16.3725,
    "lat": 48.2085,
    "grid_lon": 16.3717,
    "grid_lat": 48.213,
    "elevation_m": 171.4,
    "grid_elevation_m": 178.9
  },
  "model": "GeoSphere AROME nwp-v2-1h-1km",
  "reference_time": "2026-11-08T06:00:00Z",
  "interval": "1h",
  "valid_until": "2026-11-08T12:00:00Z",
  "ensemble": {
    "model": "GeoSphere ensemble-v2-1h-1km",
    "reference_time": "2026-11-08T03:00:00Z",
    "valid_until": "2026-11-09T09:00:00Z",
    "percentiles": [
      10,
      50,
      90
    ],
    "status": "current"
  },
  "hours": [
    {
      "time": "2026-11-08T12:00:00Z",
      "temperature_c": 10.1,
      "precipitation_mm": 0,
      "wind_speed_ms": 3.4,
      "wind_gust_ms": 7.2,
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
          "p50": 10.2,
          "p90": 11.6
        },
        "wind_gust_ms": {
          "p10": 5.1,
          "p50": 7.4,
          "p90": 10.2
        },
        "precipitation_mm": {
          "p10": 0,
          "p50": 0,
          "p90": 0.3
        }
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
| `503` | Data temporarily unavailable |

## Precipitation nowcast for a point (15 min steps, up to 3 h after the analysis)

`GET /v1/weather/nowcast/{coordinates}`

GeoSphere INCA nowcast nowcast-v1-15min-1km (a new analysis every 15 min), bilinear to the point. The horizon ends 3 h after `reference_time` (the analysis); because a run is published 20 to 40 minutes after its analysis time, the steps reach about 2 to 2.5 h ahead of now, see `valid_until`.

`precipitation_starts_in_min` / `precipitation_ends_in_min` are derived from the 15-minute steps (wet from 0.1 mm/h); `null` when nothing changes within the horizon. Outside 404 `outside-coverage`; unreadable coordinates 400 `invalid-coordinates`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

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
    "lat": 48.2085,
    "grid_lon": 16.37303,
    "grid_lat": 48.20932,
    "elevation_m": 171.4,
    "grid_elevation_m": 175.2
  },
  "model": "GeoSphere INCA nowcast-v1-15min-1km",
  "reference_time": "2026-11-08T10:45:00Z",
  "interval": "15min",
  "valid_until": "2026-11-08T13:45:00Z",
  "precipitation_now": false,
  "precipitation_starts_in_min": 30,
  "precipitation_ends_in_min": null,
  "steps": [
    {
      "time": "2026-11-08T11:15:00Z",
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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

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

param: temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall (hourly, AROME) or rr_nowcast, pt_nowcast (precipitation radar nowcast, 15 min steps). `mapx:time.times` lists the available hours (UTC), `mapx:time.template` has the placeholder `{time}` and the model run as `run` (tiles of the current run can be cached for a day); `tiles` uses `now`.

`maxzoom` is 12 (1 km data); the tile endpoint also serves z13 and z14 (smoothed). Legend values in `mapx:legend` are in `unit`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

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
  "name": "mapx weather: Air temperature 2 m",
  "tiles": [
    "https://api.mapx.at/v1/weather/tiles/temperature/now/{z}/{x}/{y}.png"
  ],
  "minzoom": 0,
  "maxzoom": 12,
  "mapx:time": {
    "reference_time": "2026-11-08T06:00:00Z",
    "default": "now",
    "times": [
      "2026-11-08T10:00:00Z",
      "2026-11-08T11:00:00Z"
    ],
    "template": "https://api.mapx.at/v1/weather/tiles/temperature/{time}/{z}/{x}/{y}.png?run=2026-11-08T06:00:00Z"
  },
  "mapx:legend": {
    "unit": "°C",
    "stops": [
      {
        "value": -25,
        "color": "rgba(88,40,140,0.78)"
      }
    ]
  }
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Weather raster tile (PNG 256, transparent outside the data)

`GET /v1/weather/tiles/{param}/{time}/{z}/{x}/{y}.png`

Color ramp per parameter (see legend in the TileJSON). time: `now` or a time such as `2026-11-07T20:00:00Z` (a UTC offset such as `+01:00` is allowed, URL-encode `+` as `%2B`); it snaps to the nearest step within 30 minutes, the step actually used is in the header `X-Weather-Time`.

Cache: `max-age=86400` for a fixed time with the current `run`, otherwise 300 s (120 s for the nowcast), because the content of a fixed time changes with the next model run. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `param` | path | string | yes | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast, pt_nowcast |
| `time` | path | string | yes | – | now or 2026-11-07T20:00:00Z (nowcast: 15 min steps such as 2026-11-07T20:15:00Z) |
| `z` | path | integer | yes | – | 0 to 14 (TileJSON maxzoom 12) |
| `x` | path | integer | yes | – | Column |
| `y` | path | string | yes | – | Row, with .png |
| `run` | query | string |  | – | Model run (`mapx:time.reference_time`) as in the TileJSON template; only affects caching |

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

**Example response**: `image/png` · PNG; headers X-Weather-Reference-Time (model run), X-Weather-Time (time step shown)

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

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

Active and upcoming official weather warnings (up to 4 days ahead) as GeoJSON MultiPolygons in WGS84, one feature per warning (merged municipalities). `type`: wind, rain, snow, black_ice, thunderstorm, heat, cold; `level`: 1 yellow, 2 orange, 3 red; `text`, `impact` and `recommendation` (each with `de` and `en`) are optional: they are present only when GeoSphere has published a text for the warning and are often missing.

Fetched centrally every 5 minutes, never per request. Without `bbox` and `lon`/`lat` all warnings are returned. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lon` | query | string |  | – | Longitude: only warnings that contain the point (with lat) |
| `lat` | query | string |  | – | Latitude |
| `bbox` | query | string |  | – | west,south,east,north in degrees (WGS84): only warnings whose area (polygon, not just its bounding box) touches the box |
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

**Example response** (shortened: lists show only their first entries)

```json
{
  "type": "FeatureCollection",
  "updated": "2026-11-08T08:00:00Z",
  "attribution": "mapx.at; Warnungen: GeoSphere Austria, CC BY 4.0 (https://warnungen.zamg.at)",
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
              ]
            ]
          ]
        ]
      },
      "properties": {
        "id": "w4149c1v1",
        "type": "wind",
        "type_id": 1,
        "level": 2,
        "level_name": "orange",
        "onset": "2026-11-08T12:00:00Z",
        "expires": "2026-11-09T04:00:00Z",
        "municipalities": 1,
        "municipality_ids": [
          "70101"
        ],
        "text": {
          "de": "Orange Sturmwarnung",
          "en": "Orange storm warning"
        },
        "impact": {
          "de": "Windspitzen über 70 km/h",
          "en": "Wind speeds exceeding 70 km/h"
        },
        "recommendation": {
          "de": null,
          "en": null
        }
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
| `503` | Data temporarily unavailable |

## Playground

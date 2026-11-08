# Weather API

Current observations, hourly forecast and weather tiles (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

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

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/weather/current?lon=16.3725&lat=48.2085&limit=3" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/weather/current?lon=16.3725&lat=48.2085&limit=3");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/weather/current?lon=16.3725&lat=48.2085&limit=3")
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

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/weather/forecast?lon=16.3725&lat=48.2085&hours=6" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/weather/forecast?lon=16.3725&lat=48.2085&hours=6");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/weather/forecast?lon=16.3725&lat=48.2085&hours=6")
data = res.json()
```

</TabItem>
</Tabs>

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
      "pressure_msl_hpa": 1008.3
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
| `503` | Data temporarily unavailable |

## TileJSON of weather tiles with time dimension

`GET /v1/weather/tiles/{param}.json`

param: temperature, precipitation, wind, humidity, cloud, pressure. `mapx:time.times` lists the available hours (UTC), `mapx:time.template` has the placeholder `{time}`; `tiles` uses `now`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `param` | path | string | yes | – | temperature, precipitation, wind, humidity, cloud or pressure, with .json |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/weather/tiles/temperature.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/weather/tiles/temperature.json");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/weather/tiles/temperature.json")
data = res.json()
```

</TabItem>
</Tabs>

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
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Weather raster tile (PNG 256, transparent outside the data)

`GET /v1/weather/tiles/{param}/{time}/{z}/{x}/{y}.png`

Color ramp per parameter (see legend in the TileJSON). time: `now` or a UTC hour such as `2026-11-07T20:00Z`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `param` | path | string | yes | – | temperature, precipitation, wind, humidity, cloud, pressure |
| `time` | path | string | yes | – | now or 2026-11-07T20:00Z |
| `z` | path | integer | yes | – | 0 to 14 |
| `x` | path | integer | yes | – | Column |
| `y` | path | string | yes | – | Row, with .png |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/weather/tiles/temperature/now/6/34/22.png" -o result.png
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/weather/tiles/temperature/now/6/34/22.png");
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/weather/tiles/temperature/now/6/34/22.png")
open("result.png", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Example response**: `image/png` · PNG; header X-Weather-Reference-Time

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Playground

<Playground group="weather" lang="en" />
# Weather API

Aktuelle Messwerte, Stundenvorhersage, Niederschlags-Nowcast, Warnungen und Wetterkacheln (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Messwerte der nächsten Stationen, Stundenvorhersage für einen Punkt und Rasterkacheln für ein Karten-Overlay. Quelle: GeoSphere Austria, CC BY 4.0.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Karten](/de/guides/maps/)

## Current observations of the nearest TAWES stations

`GET /v1/weather/current`

GeoSphere Austria station network, 10-minute values (temperature, humidity, wind, gust, precipitation, pressure). Nearest station first. Values come from a local cache, never fetched per request. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lon` | query | string | ja | – | Longitude (also `lng`), WGS84 |
| `lat` | query | string | ja | – | Latitude |
| `limit` | query | string |  | `3` | 1 to 10, default 3 |
| `radius` | query | string |  | `50` | Search radius in km, default 50 |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Hourly point forecast (AROME, 1 km)

`GET /v1/weather/forecast`

GeoSphere Austria model nwp-v2-1h-1km, bilinear to the point, from the current hour. Austria plus margin. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lon` | query | string | ja | – | Longitude (also `lng`), WGS84 |
| `lat` | query | string | ja | – | Latitude |
| `hours` | query | string |  | `24` | 1 to 60, default 24 |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## TileJSON of weather tiles with time dimension

`GET /v1/weather/tiles/{param}.json`

param: temperature, precipitation, wind, humidity, cloud, pressure, or the air quality layers aqi (daily index), no2, o3, pm10, pm25 (near-surface, EEA limits) and dust (Saharan dust load). `mapx:time.times` lists the available hours (UTC), `mapx:time.template` has the placeholder `{time}`; `tiles` uses `now`.

<details>
<summary>Mehr</summary>

Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `param` | path | string | ja | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25 or dust, with .json |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Weather raster tile (PNG 256, transparent outside the data)

`GET /v1/weather/tiles/{param}/{time}/{z}/{x}/{y}.png`

Color ramp per parameter (see legend in the TileJSON). time: `now` or a UTC hour such as `2026-11-07T20:00Z`. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `param` | path | string | ja | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust |
| `time` | path | string | ja | – | now or 2026-11-07T20:00Z |
| `z` | path | integer | ja | – | 0 to 14 |
| `x` | path | integer | ja | – | Column |
| `y` | path | string | ja | – | Row, with .png |

**Beispiel**

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

**Beispielantwort**: `image/png` · PNG; header X-Weather-Reference-Time

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Official weather warnings (GeoSphere Austria)

`GET /v1/weather/warnings`

Active and upcoming official weather warnings (up to 4 days ahead) as GeoJSON MultiPolygons in WGS84, one feature per warning (merged municipalities). `type`: wind, rain, snow, black_ice, thunderstorm, heat, cold; `level`: 1 yellow, 2 orange, 3 red; `text`, `impact` and `recommendation` carry `de` and `en`.

<details>
<summary>Mehr</summary>

Fetched centrally every 5 minutes, never per request. Without `bbox` and `lon`/`lat` all warnings are returned. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lon` | query | string |  | – | Longitude: only warnings that contain the point (with lat) |
| `lat` | query | string |  | – | Latitude |
| `bbox` | query | string |  | – | west,south,east,north in degrees (WGS84): only warnings touching the box |
| `type` | query | string |  | – | wind, rain, snow, black_ice, thunderstorm, heat or cold |
| `min_level` | query | string |  | – | 1 (yellow, default) to 3 (red) |
| `active` | query | string |  | – | now: only warnings valid right now; all (default): also upcoming |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/weather/warnings?bbox=9.4,46.3,17.3,49.1&min_level=1" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/weather/warnings?bbox=9.4,46.3,17.3,49.1&min_level=1");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/weather/warnings?bbox=9.4,46.3,17.3,49.1&min_level=1")
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

<Playground group="weather" lang="de" />
# Air Quality API

Air quality index, pollutants and Saharan dust for a point (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Daily air quality index and hourly nitrogen dioxide, ozone, PM10 and PM2.5 near the surface plus the Saharan dust load for a point, from the GeoSphere chemical weather model (3 km). The index follows the EEA European Air Quality Index (1 good to 6 extremely poor; SO2 is not provided by the model). Map layers: the weather tiles `aqi`, `no2`, `o3`, `pm10`, `pm25` and `dust` (see [Weather API](/api/weather/)). Source: GeoSphere Austria, CC BY 4.0.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Maps](/guides/maps/)

## Air quality index and pollutants for a point

`GET /v1/air-quality/{coordinate}`

GeoSphere chemical weather model (3 km) bilinear to the point: daily index `aqi` (3 days) and hourly NO2, O3, PM10, PM2.5 near the surface in ug/m3 with an hourly index (worst level by the EEA European Air Quality Index limits; SO2 is not provided by the model) plus the Saharan dust load (mg/m2, 0.2 degree grid).

<details>
<summary>More</summary>

Austria and surroundings. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinate` | path | string | yes | – | lon,lat, e.g. 16.37,48.21 |
| `hours` | query | integer |  | `24` | 1 to 72, default 24 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/air-quality/16.3725,48.2085?hours=6" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/air-quality/16.3725,48.2085?hours=6");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/air-quality/16.3725,48.2085?hours=6")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "location": {
    "lon": 16.37,
    "lat": 48.21
  },
  "model": "GeoSphere chemical weather forecast",
  "reference_time": "2026-11-08T00:00Z",
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
      "time": "2026-11-08T12:00Z",
      "index": 2,
      "category": "fair",
      "no2_ugm3": 18.2,
      "o3_ugm3": 45.1,
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Playground

<Playground group="air-quality" lang="en" />
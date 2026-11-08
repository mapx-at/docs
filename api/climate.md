# Climate API

Solar potential, climate normals, heat days, snow and drought at a point (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Climate of a point: yearly solar radiation and sunshine hours, temperature and precipitation normals with trend, hot and frost days, snow days per winter and the drought index. Source: GeoSphere Austria, CC BY 4.0. The same object is `properties.climate` in the [parcel dossier](/api/tilequery/).

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Parcels](/guides/parcels/)

## Climate at a point (solar potential, normals, heat days, snow, drought)

`GET /v1/climate/{coordinate}`

Climate of one point from open data of GeoSphere Austria: solar potential (yearly global radiation in kWh/m2 and sunshine hours, mean of the last 3 complete years, 100 m), temperature and precipitation (normal 1991-2020, last 10 years, trend, monthly normals, 1 km), hot/summer/frost/ice days and tropical nights per year, snow days per winter and the current snow depth, and the drought index SPEI (90 and 365 days, latest value with class).

<details>
<summary>More</summary>

Each section has `source` (dataset, cell, last day of data) and `retrieved_at`. Values are derived per grid cell and cached; the first request for a new cell fetches them from the source and can take 10 to 20 seconds (`sections` limits the work), later requests come from the cache. The same object is part of the parcel dossier (`properties.climate`). Outside Austria 404; 503 while the source is unavailable or rate limited. Data source: GeoSphere Austria (APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE), CC BY 4.0 (attribution required).

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `coordinate` | path | string | yes | – | lon,lat or lon,lat.json |
| `sections` | query | string |  | `all` | Comma-separated sections to return (default: all) |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/climate/15.3294,48.2272?sections=solar,snow", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

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
    "sunshine_hours_year": 2949.5,
    "relative_sunshine_pct": 60.3,
    "by_year": [
      {
        "year": 2025,
        "global_radiation_kwh_m2": 1310.8,
        "sunshine_hours": 2958.4
      }
    ],
    "source": {
      "provider": "GeoSphere Austria",
      "dataset": "apolis-v2-1d-100m",
      "name": "APOLIS solar radiation",
      "license": "CC BY 4.0",
      "resolution_m": 100,
      "cell_center": [
  …
```

| Field | Type | Description |
|---|---|---|
| `location` | array | [longitude, latitude] (WGS84), optional height. |
| `status` | string | `partial`: some sections are still being loaded (parcel dossier: retry after `retry_after_s`) |
| `solar` | object,null | Solar potential: yearly sums, mean of the last 3 complete years (APOLIS, 100 m) |
| `temperature` | object,null | Air temperature (SPARTACUS, 1 km) |
| `precipitation` | object,null | Precipitation and sunshine (SPARTACUS, 1 km) |
| `temperature_extremes` | object,null | Hot, summer, frost and ice days per year, tropical nights (SPARTACUS daily data, 1 km); normal period 1991-2020 and last 10 complete years |
| `snow` | object,null | Snow cover (SNOWGRID-CL model, 1 km) |
| `drought` | object,null | Drought index SPEI (WINFORE, 1 km), latest value |

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Playground

<Playground group="climate" lang="en" />
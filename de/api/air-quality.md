# Air Quality API

Luftqualitätsindex, Schadstoffe und Saharastaub für einen Punkt (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Täglicher Luftqualitätsindex und stündlich Stickstoffdioxid, Ozon, PM10 und PM2.5 bodennah sowie die Saharastaub-Fracht für einen Punkt, aus dem Chemiewettermodell der GeoSphere (3 km). Der Index folgt dem Europäischen Luftqualitätsindex der EEA (1 gut bis 6 sehr schlecht; SO2 liefert das Modell nicht). Kartenebenen: die Wetterkacheln `aqi`, `no2`, `o3`, `pm10`, `pm25` und `dust` (siehe [Weather API](/de/api/weather/)). Quelle: GeoSphere Austria, CC BY 4.0.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Karten](/de/guides/maps/)

## Air quality index and pollutants for a point

`GET /v1/air-quality/{coordinate}`

GeoSphere chemical weather model (3 km) bilinear to the point: daily index `aqi` (3 days) and hourly NO2, O3, PM10, PM2.5 near the surface in ug/m3 with an hourly index (worst level by the EEA European Air Quality Index limits; SO2 is not provided by the model) plus the Saharan dust load (mg/m2, 0.2 degree grid).

<details>
<summary>Mehr</summary>

Austria and surroundings. Data source: GeoSphere Austria, CC BY 4.0 (attribution required).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat, e.g. 16.37,48.21 |
| `hours` | query | integer |  | `24` | 1 to 72, default 24 |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

<Playground group="air-quality" lang="de" />
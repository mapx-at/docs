# Luftqualität API für Österreich

Source: https://docs.mapx.at/de/api/air-quality/

> Luftqualitätsindex, Schadstoffe und Saharastaub für einen Punkt in Österreich von GeoSphere Austria: Endpunkte, Parameter und Beispielanfragen.

Täglicher Luftqualitätsindex und stündlich Stickstoffdioxid, Ozon, PM10 und PM2.5 bodennah sowie die Saharastaub-Fracht für einen Punkt, aus dem Chemiewettermodell der GeoSphere (3 km). Der Index folgt dem Europäischen Luftqualitätsindex der EEA (1 gut bis 6 sehr schlecht; SO2 liefert das Modell nicht). Kartenebenen: die Wetterkacheln `aqi`, `no2`, `o3`, `pm10`, `pm25` und `dust` (siehe [Weather API](https://docs.mapx.at/de/api/weather.md)). Quelle: GeoSphere Austria, CC BY 4.0.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Karten](https://docs.mapx.at/de/guides/maps.md)

## Luftqualitätsindex und Schadstoffe für einen Punkt {#air-quality-index-and-pollutants-for-a-point}

`GET /v1/air-quality/{coordinate}`

Chemisches Wettermodell von GeoSphere (3 km), bilinear auf den Punkt interpoliert: Tagesindex `aqi` (3 Tage) und stündlich NO2, O3, PM10, PM2.5 in Bodennähe in ug/m3 mit Stundenindex (schlechteste Stufe nach den Grenzen des European Air Quality Index der EEA; SO2 liefert das Modell nicht), dazu die Saharastaubbelastung (mg/m2, Raster 0,2 Grad).

Österreich und Umgebung. Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat, z. B. 16.37,48.21 |
| `hours` | query | integer |  | `24` | 1 bis 72, Standard 24; der tägliche Lauf (00 UTC) reicht 48 h weit, weniger Stunden als verlangt sind mit `truncated: true` gekennzeichnet |

**Beispiel**

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

**Beispielantwort**

```json
{
  "location": [
    16.37,
    48.21
  ],
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
  "attribution": "mapx.at; GeoSphere Austria (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

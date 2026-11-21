# Wetter-API: Vorhersage und Nowcast

Source: https://docs.mapx.at/de/api/weather/

> Wetter-API für Österreich: aktuelle Messwerte, Stundenvorhersage, Niederschlags-Nowcast, Warnungen und Wetterkacheln von GeoSphere Austria.

Messwerte der nächsten Stationen, Stundenvorhersage für einen Punkt und Rasterkacheln für ein Karten-Overlay. Quelle: GeoSphere Austria, CC BY 4.0.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Karten](https://docs.mapx.at/de/guides/maps.md)

## Aktuelle Messwerte der nächsten TAWES-Stationen {#current-observations-of-the-nearest-tawes-stations}

`GET /v1/weather/current`

Messnetz von GeoSphere Austria, 10-Minuten-Werte (Temperatur, Luftfeuchte, Wind, Böe, Niederschlag, Luftdruck). Die nächste Station zuerst. Die Werte kommen aus einem lokalen Cache und werden nie je Anfrage abgerufen. Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lon` | query | string | ja | – | Länge (auch `lng`), WGS84 |
| `lat` | query | string | ja | – | Breite |
| `limit` | query | string |  | `3` | 1 bis 10, Standard 3 |
| `radius` | query | string |  | `50` | Suchradius in km, Standard 50 |

**Beispiel**

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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Stündliche Punktvorhersage (AROME, 1 km) {#hourly-point-forecast-arome-1-km}

`GET /v1/weather/forecast`

Modell nwp-v2-1h-1km von GeoSphere Austria, bilinear auf den Punkt interpoliert, ab der aktuellen Stunde. Österreich mit Rand. Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lon` | query | string | ja | – | Länge (auch `lng`), WGS84 |
| `lat` | query | string | ja | – | Breite |
| `hours` | query | string |  | `24` | 1 bis 60, Standard 24 |
| `ensemble` | query | string |  | – | true: `uncertainty` (p10/p50/p90 des Ensembles für Temperatur, Böe, Niederschlag) zu jeder Stunde ergänzen |

**Beispiel**

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

**Beispielantwort**

```json
{
  "location": {
    "lon": 16.3725,
    "lat": 48.2085,
    "grid_lon": 16.3717,
    "grid_lat": 48.213
  },
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
  …
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

## Niederschlags-Nowcast für einen Punkt (15-Minuten-Schritte, 3 h) {#precipitation-nowcast-for-a-point-15-min-steps-3-h}

`GET /v1/weather/nowcast/{coordinates}`

INCA-Nowcast nowcast-v1-15min-1km von GeoSphere, bilinear auf den Punkt interpoliert. `precipitation_starts_in_min` / `precipitation_ends_in_min` werden aus den 15-Minuten-Schritten abgeleitet (nass ab 0,1 mm/h); `null`, wenn sich im Vorhersagezeitraum nichts ändert.

Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | path | string | ja | – | lon,lat, z. B. 16.3725,48.2085 |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## TileJSON der Wetterkacheln mit Zeitdimension {#tilejson-of-weather-tiles-with-time-dimension}

`GET /v1/weather/tiles/{param}.json`

param: temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall (stündlich, AROME) oder rr_nowcast, pt_nowcast (Niederschlagsradar-Nowcast, 15-Minuten-Schritte). `mapx:time.times` listet die verfügbaren Stunden (UTC), `mapx:time.template` enthält den Platzhalter `{time}`; `tiles` verwendet `now`.

Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `param` | path | string | ja | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast oder pt_nowcast, mit .json |

**Beispiel**

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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Wetter-Rasterkachel (PNG 256, außerhalb der Daten transparent) {#weather-raster-tile-png-256-transparent-outside-the-data}

`GET /v1/weather/tiles/{param}/{time}/{z}/{x}/{y}.png`

Farbskala je Parameter (siehe Legende im TileJSON). time: `now` oder eine UTC-Stunde wie `2026-11-07T20:00Z`. Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `param` | path | string | ja | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast, pt_nowcast |
| `time` | path | string | ja | – | now oder 2026-11-07T20:00Z (Nowcast: 15-Minuten-Schritte wie 2026-11-07T20:15Z) |
| `z` | path | integer | ja | – | 0 bis 14 |
| `x` | path | integer | ja | – | Spalte |
| `y` | path | string | ja | – | Zeile, mit .png |

**Beispiel**

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

**Beispielantwort**: `image/png` · PNG; Header X-Weather-Reference-Time

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Amtliche Wetterwarnungen (GeoSphere Austria) {#official-weather-warnings-geosphere-austria}

`GET /v1/weather/warnings`

Aktive und kommende amtliche Wetterwarnungen (bis zu 4 Tage im Voraus) als GeoJSON-MultiPolygone in WGS84, ein Objekt je Warnung (zusammengefasste Gemeinden). `type`: wind, rain, snow, black_ice, thunderstorm, heat, cold; `level`: 1 gelb, 2 orange, 3 rot; `text`, `impact` und `recommendation` (jeweils mit `de` und `en`) sind optional: Sie sind nur vorhanden, wenn GeoSphere zur Warnung einen Text veröffentlicht hat, und fehlen oft.

Zentral alle 5 Minuten abgerufen, nie je Anfrage. Ohne `bbox` und `lon`/`lat` kommen alle Warnungen. Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lon` | query | string |  | – | Länge: nur Warnungen, die den Punkt enthalten (mit lat) |
| `lat` | query | string |  | – | Breite |
| `bbox` | query | string |  | – | west,south,east,north in Grad (WGS84): nur Warnungen, die das Rechteck berühren |
| `type` | query | string |  | – | wind, rain, snow, black_ice, thunderstorm, heat oder cold |
| `min_level` | query | string |  | – | 1 (gelb, Standard) bis 3 (rot) |
| `active` | query | string |  | – | now: nur jetzt gültige Warnungen; all (Standard): auch kommende |

**Beispiel**

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

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "updated": "2026-11-08T08:00Z",
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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

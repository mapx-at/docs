# Wetter-API: Vorhersage und Nowcast

Source: https://docs.mapx.at/de/api/weather/

> Wetter-API für Österreich: aktuelle Messwerte, Stundenvorhersage, Niederschlags-Nowcast, Warnungen und Wetterkacheln von GeoSphere Austria.

Messwerte der nächsten Stationen, Stundenvorhersage für einen Punkt und Rasterkacheln für ein Karten-Overlay. Quelle: GeoSphere Austria, CC BY 4.0.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Karten](https://docs.mapx.at/de/guides/maps.md)

## Aktuelle Messwerte der nächsten TAWES-Stationen {#current-observations-of-the-nearest-tawes-stations}

`GET /v1/weather/current`

Messnetz der GeoSphere Austria, 10-Minuten-Werte (Temperatur, Feuchte, Wind, Böe, Niederschlag, Luftdruck). Nächste Station zuerst; `altitude` der Station mit `elevation` des Punkts vergleichen (Geländehöhe aus dem mapx-DGM, 1 m). Die Werte kommen aus einem lokalen Zwischenspeicher, nie je Anfrage von der Quelle.

Punkte außerhalb Österreichs und der Umgebung: 404 `outside-coverage`. Datenquelle: GeoSphere Austria, CC BY 4.0 (Namensnennung erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lng` | query | string | ja | – | Geografische Länge, WGS84 |
| `lat` | query | string | ja | – | Breite |
| `limit` | query | string |  | `3` | 1 bis 10, Standard 3 |
| `radius` | query | string |  | `50000` | Suchradius in Metern, 1 bis 600000, Standard 50000 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/current?lng=16.3725&lat=48.2085&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/weather/current?lng=16.3725&lat=48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/current?lng=16.3725&lat=48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "location": [
    16.3725,
    48.2085
  ],
  "elevation": 171.4,
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
        "altitude": 198,
        "distance": 4600,
        "time": "2026-11-08T11:50:00Z",
        "temperature_c": 9.4,
        "humidity_pct": 71,
        "wind_speed_ms": 3.1,
        "wind_gust_ms": 5.8,
        "wind_direction": 290,
        "precipitation_mm_10min": 0,
        "pressure_hpa": 1002.1,
        "pressure_msl_hpa": 1018.6
      }
    }
  ],
  "time": "2026-11-08T11:50:00Z",
  "attribution": "mapx.at; GeoSphere Austria (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Stündliche Punktvorhersage (AROME, 1 km) {#hourly-point-forecast-arome-1-km}

`GET /v1/weather/forecast`

Modell nwp-v2-1h-1km der GeoSphere Austria (Läufe alle 3 h), bilinear auf den Punkt, ab der aktuellen Stunde. Österreich mit Rand; außerhalb 404 `outside-coverage`. Ein neuer Lauf wird mit seinen ersten 24 Stunden veröffentlicht und im Hintergrund verlängert; bis dahin kommen die späteren Stunden aus dem vorigen Lauf (`previous_run`).

`weather_symbol.code` ist das GeoSphere-Symbol 1 bis 32: 1 klar, 2 überwiegend klar, 3 teils bewölkt, 4 überwiegend bewölkt, 5 bedeckt, 6–7 Nebel, 8–13 Regen, 14–19 Schnee, 20–25 Schneeregen, 26–32 Gewitter (ohne Tag/Nacht-Unterscheidung). Datenquelle: GeoSphere Austria, CC BY 4.0 (Namensnennung erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lng` | query | string | ja | – | Geografische Länge, WGS84 |
| `lat` | query | string | ja | – | Breite |
| `hours` | query | string |  | `24` | 1 bis 48, Standard 24; größere Werte ergeben 400 |
| `ensemble` | query | string |  | – | true oder false (Standard). true: `uncertainty` (p10/p50/p90 des Ensembles für Temperatur, Böe, Niederschlag) zu den Stunden, die das Ensemble abdeckt (die nächsten 24 h); siehe `ensemble.status` |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/forecast?lng=16.3725&lat=48.2085&hours=6&ensemble=true" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/weather/forecast?lng=16.3725&lat=48.2085&hours=6&ensemble=true", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/weather/forecast?lng=16.3725&lat=48.2085&hours=6&ensemble=true", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "location": [
    16.3725,
    48.2085
  ],
  "grid_location": [
    16.3717,
    48.213
  ],
  "elevation": 171.4,
  "grid_elevation": 178.9,
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
  "hourly": [
    {
      "time": "2026-11-08T12:00:00Z",
      "temperature_c": 10.1,
      "precipitation_mm": 0,
      "wind_speed_ms": 3.4,
      "wind_gust_ms": 7.2,
      "wind_direction": 285,
      "humidity_pct": 68,
      "cloud_cover_pct": 40,
      "pressure_msl_hpa": 1008.3,
      "weather_symbol": {
        "code": 3,
        "text": "Partly cloudy",
        "icon": "partly-cloudy"
      },
      "snow_limit": 1200,
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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Niederschlags-Nowcast für einen Punkt (15-Minuten-Schritte, bis 3 h nach der Analyse) {#precipitation-nowcast-for-a-point-15-min-steps-up-to-3-h-after-the-analysis}

`GET /v1/weather/nowcast/{coordinates}`

INCA-Nowcast nowcast-v1-15min-1km der GeoSphere Austria (alle 15 min eine neue Analyse), bilinear auf den Punkt. Der Horizont endet 3 h nach `reference_time` (der Analyse); weil ein Lauf 20 bis 40 Minuten nach seiner Analysezeit erscheint, reichen die Schritte etwa 2 bis 2,5 h über jetzt hinaus, siehe `valid_until`.

`precipitation_starts_in_min` / `precipitation_ends_in_min` werden aus den 15-Minuten-Schritten abgeleitet (nass ab 0,1 mm/h); `null`, wenn sich im Horizont nichts ändert. Außerhalb 404 `outside-coverage`; unlesbare Koordinaten 400 `invalid-coordinates`. Datenquelle: GeoSphere Austria, CC BY 4.0 (Namensnennung erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | path | string | ja | – | lng,lat, z. B. 16.3725,48.2085 |

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
  "location": [
    16.3725,
    48.2085
  ],
  "grid_location": [
    16.37303,
    48.20932
  ],
  "elevation": 171.4,
  "grid_elevation": 175.2,
  "model": "GeoSphere INCA nowcast-v1-15min-1km",
  "reference_time": "2026-11-08T10:45:00Z",
  "interval": "15min",
  "valid_until": "2026-11-08T13:45:00Z",
  "precipitation_now": false,
  "precipitation_starts_in_min": 30,
  "precipitation_ends_in_min": null,
  "minutely": [
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
      "wind_direction": 290
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## TileJSON der Wetterkacheln mit Zeitdimension {#tilejson-of-weather-tiles-with-time-dimension}

`GET /v1/weather/tiles/{param}.json`

param: temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall (stündlich, AROME) oder rr_nowcast, pt_nowcast (Niederschlagsradar-Nowcast, 15-Minuten-Schritte). `mapx:time.times` listet die verfügbaren Stunden (UTC), `mapx:time.template` hat den Platzhalter `{time}` und den Modelllauf als `run` (Kacheln des aktuellen Laufs dürfen einen Tag zwischengespeichert werden); `tiles` nutzt `now`.

`maxzoom` ist 12 (1-km-Daten); der Kachel-Endpunkt liefert auch z13 und z14 (geglättet). Die Legendenwerte in `mapx:legend` sind in `unit`. Datenquelle: GeoSphere Austria, CC BY 4.0 (Namensnennung erforderlich).

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

Farbskala je Parameter (siehe Legende im TileJSON). time: `now` oder eine Zeit wie `2026-11-07T20:00:00Z` (ein UTC-Offset wie `+01:00` ist erlaubt, `+` in der URL als `%2B`); sie rastet auf den nächsten Schritt innerhalb von 30 Minuten ein, der tatsächlich verwendete Schritt steht im Header `X-Weather-Time`.

Cache: `max-age=86400` für eine feste Zeit mit dem aktuellen `run`, sonst 300 s (Nowcast 120 s), weil sich der Inhalt einer festen Zeit mit dem nächsten Modelllauf ändert. Datenquelle: GeoSphere Austria, CC BY 4.0 (Namensnennung erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `param` | path | string | ja | – | temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast, pt_nowcast |
| `time` | path | string | ja | – | now oder 2026-11-07T20:00:00Z (Nowcast: 15-Minuten-Schritte wie 2026-11-07T20:15:00Z) |
| `z` | path | integer | ja | – | 0 bis 14 (TileJSON-maxzoom 12) |
| `x` | path | integer | ja | – | Spalte |
| `y` | path | string | ja | – | Zeile, mit .png |
| `run` | query | string |  | – | Modelllauf (`mapx:time.reference_time`) wie im TileJSON-Template; wirkt nur auf das Caching |

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

**Beispielantwort**: `image/png` · PNG; Header X-Weather-Reference-Time (Modelllauf), X-Weather-Time (gezeigter Zeitschritt)

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Wetterwarnungen (GeoSphere Austria) {#weather-warnings-geosphere-austria}

`GET /v1/weather/warnings`

Aktive und kommende Wetterwarnungen von GeoSphere Austria (bis zu 4 Tage im Voraus) als GeoJSON-MultiPolygone in WGS84, ein Objekt je Warnung (zusammengefasste Gemeinden). `type`: wind, rain, snow, black_ice, thunderstorm, heat, cold; `level`: 1 gelb, 2 orange, 3 rot; `text`, `impact` und `recommendation` (jeweils mit `de` und `en`) sind optional: Sie sind nur vorhanden, wenn GeoSphere zur Warnung einen Text veröffentlicht hat, und fehlen oft.

Zentral alle 5 Minuten abgerufen, nie je Anfrage. Ohne `bbox` und `lng`/`lat` kommen alle Warnungen. Datenquelle: GeoSphere Austria, CC BY 4.0 (Quellenangabe erforderlich).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lng` | query | string |  | – | Geografische Länge: nur Warnungen, die den Punkt enthalten (mit lat) |
| `lat` | query | string |  | – | Breite |
| `bbox` | query | string |  | – | west,süd,ost,nord in Grad (WGS84): nur Warnungen, deren Gebiet (Polygon, nicht nur dessen Hüllrechteck) das Rechteck berührt |
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

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "type": "FeatureCollection",
  "updated": "2026-11-08T08:00:00Z",
  "attribution": "mapx.at; GeoSphere Austria (CC BY 4.0)",
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

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

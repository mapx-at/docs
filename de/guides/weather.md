# Wetter-Guide: Messwerte, Vorhersage, Warnungen

Source: https://docs.mapx.at/de/guides/weather/

> Wetter- und Umweltdaten für einen Punkt in Österreich: Stationsmesswerte, Stundenvorhersage, Niederschlags-Nowcast, Warnungen, Wetterkacheln, Luftqualität und Klima.

Alle Wetter- und Umweltdaten stammen von GeoSphere Austria (CC BY 4.0). Anfragen brauchen einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); Punktabfragen laufen in der Zone `search`, Wetterkacheln in der Zone `weather`. Zeiten sind UTC mit `Z`, Einheiten folgen den [Namensregeln](https://docs.mapx.at/de/reference/conventions.md) (`temperature_c`, `precipitation_mm`, `wind_speed_ms`; `wind_direction` in Grad ohne Suffix).

| Aufgabe | Endpunkt |
|---|---|
| Messwerte der nächsten Stationen | [`/v1/weather/current`](https://docs.mapx.at/de/api/weather.md) |
| Stundenvorhersage für einen Punkt (AROME, 1 km) | [`/v1/weather/forecast`](https://docs.mapx.at/de/api/weather.md) |
| Niederschlag der nächsten 3 Stunden (15-Minuten-Schritte) | [`/v1/weather/nowcast/{coordinates}`](https://docs.mapx.at/de/api/weather.md) |
| Wetterwarnungen von GeoSphere Austria | [`/v1/weather/warnings`](https://docs.mapx.at/de/api/weather.md) |
| Wetter- und Radarebene auf der Karte | [`/v1/weather/tiles/{param}.json`](https://docs.mapx.at/de/api/weather.md) |
| Luftqualitätsindex und Schadstoffe | [`/v1/air-quality/{coordinate}`](https://docs.mapx.at/de/api/air-quality.md) |
| Klima eines Ortes (Sonne, Normale, Schnee, Dürre) | [`/v1/climate/{coordinate}`](https://docs.mapx.at/de/api/climate.md) |

## Messwerte und Vorhersage

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/current?lng=15.3294&lat=48.2272&limit=1" | jq '.features[0].properties | {name, temperature_c, wind_speed_ms}'
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/forecast?lng=15.3294&lat=48.2272&hours=6" | jq '.hourly[] | {time, temperature_c, precipitation_mm}'
```

`current` liefert die TAWES-Stationen nahe dem Punkt mit `distance` (Meter) und `altitude`. `forecast` liefert `hourly[]`; `location` ist der angefragte Punkt, `grid_location` die verwendete Modellzelle, beide `[lng, lat]`. Mit `ensemble=true` kommt eine Unsicherheitsspanne dazu.

## Regnet es gleich?

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/nowcast/15.3294,48.2272" | jq '{precipitation_now, precipitation_starts_in_min}'
```

Der Nowcast (INCA) hat 15-Minuten-Schritte in `minutely[]` bis drei Stunden nach der Analyse. `precipitation_starts_in_min` und `precipitation_ends_in_min` beantworten die übliche Frage direkt.

## Warnungen

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/warnings?lng=15.3294&lat=48.2272&min_level=2"
```

Eine GeoJSON-FeatureCollection mit den Warngebieten. Filter nach Punkt (`lng` und `lat`) oder `bbox`, nach `type` (`wind`, `rain`, `snow`, `black_ice`, `thunderstorm`, `heat`, `cold`) und nach `min_level` (1 gelb bis 3 rot). Den Warntext unverändert anzeigen.

## Wetter auf der Karte

`/v1/weather/tiles/{param}.json` ist eine TileJSON mit Zeitdimension (`mapx:time.times`); Parameter sind unter anderem `temperature`, `precipitation`, `wind`, `cloud`, `snowlimit`, der Radar-Nowcast `rr_nowcast` und die Luftqualitätsebenen `aqi`, `no2`, `o3`, `pm10`, `pm25`, `dust`. Fertige Beispiele: [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md) und [Niederschlagsradar](https://docs.mapx.at/de/examples/radar-animation.md).

## Luftqualität und Klima

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/air-quality/15.3294,48.2272?hours=6" | jq '.daily'
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/climate/15.3294,48.2272?sections=sunshine" | jq '.sunshine.hours_year'
```

Die Luftqualität folgt dem Europäischen Luftqualitätsindex (1 gut bis 6 sehr schlecht), täglich und stündlich. Das Klima umfasst Sonnenpotenzial, Temperatur- und Niederschlagsnormale, Sonnenstunden, Hitze- und Frosttage, Schneetage und den Dürreindex; Teile wählt `sections`.

## Quellenangabe

`mapx.at | GeoSphere Austria (CC BY 4.0)` (Feld `attribution` der Antwort) überall anzeigen, wo die Daten erscheinen, siehe [Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md).

## Weiter

- [Wetter-API](https://docs.mapx.at/de/api/weather.md), [Luftqualität-API](https://docs.mapx.at/de/api/air-quality.md), [Klima-API](https://docs.mapx.at/de/api/climate.md)
- Beispiele: [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md), [Niederschlagsradar](https://docs.mapx.at/de/examples/radar-animation.md)

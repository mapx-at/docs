# Weather guide: observations, forecast, warnings

Source: https://docs.mapx.at/guides/weather/

> Weather and environment data for a point in Austria: station observations, hourly forecast, precipitation nowcast, warnings, weather tiles, air quality and climate.

All weather and environment data come from GeoSphere Austria (CC BY 4.0). Requests need a token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); point requests are in the `search` zone, weather tiles in the `weather` zone. Times are UTC with `Z`, units follow the [naming rules](https://docs.mapx.at/reference/conventions.md) (`temperature_c`, `precipitation_mm`, `wind_speed_ms`; `wind_direction` in degrees without suffix).

| Task | Endpoint |
|---|---|
| Measured values of the nearest stations | [`/v1/weather/current`](https://docs.mapx.at/api/weather.md) |
| Hourly forecast for a point (AROME, 1 km) | [`/v1/weather/forecast`](https://docs.mapx.at/api/weather.md) |
| Precipitation in the next 3 hours (15-minute steps) | [`/v1/weather/nowcast/{coordinates}`](https://docs.mapx.at/api/weather.md) |
| Weather warnings from GeoSphere Austria | [`/v1/weather/warnings`](https://docs.mapx.at/api/weather.md) |
| Weather and radar layer on the map | [`/v1/weather/tiles/{param}.json`](https://docs.mapx.at/api/weather.md) |
| Air quality index and pollutants | [`/v1/air-quality/{coordinate}`](https://docs.mapx.at/api/air-quality.md) |
| Climate of a place (sunshine, normals, snow, drought) | [`/v1/climate/{coordinate}`](https://docs.mapx.at/api/climate.md) |

## Observations and forecast

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/current?lng=15.3294&lat=48.2272&limit=1" | jq '.features[0].properties | {name, temperature_c, wind_speed_ms}'
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/forecast?lng=15.3294&lat=48.2272&hours=6" | jq '.hourly[] | {time, temperature_c, precipitation_mm}'
```

`current` returns the TAWES stations closest to the point with `distance` (meters) and `altitude`. `forecast` returns `hourly[]`; `location` is the requested point, `grid_location` the model cell used, both `[lng, lat]`. With `ensemble=true` the forecast adds an uncertainty range.

## Will it rain soon?

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/nowcast/15.3294,48.2272" | jq '{precipitation_now, precipitation_starts_in_min}'
```

The nowcast (INCA) has 15-minute steps in `minutely[]` up to three hours after the analysis. `precipitation_starts_in_min` and `precipitation_ends_in_min` answer the usual question directly.

## Warnings

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/weather/warnings?lng=15.3294&lat=48.2272&min_level=2"
```

A GeoJSON FeatureCollection with the warning areas. Filter by point (`lng` and `lat`) or by `bbox`, by `type` (`wind`, `rain`, `snow`, `black_ice`, `thunderstorm`, `heat`, `cold`) and by `min_level` (1 yellow to 3 red). Show the warning text unchanged.

## Weather on the map

`/v1/weather/tiles/{param}.json` is a TileJSON with a time dimension (`mapx:time.times`); parameters include `temperature`, `precipitation`, `wind`, `cloud`, `snowlimit`, the radar nowcast `rr_nowcast` and the air quality layers `aqi`, `no2`, `o3`, `pm10`, `pm25`, `dust`. Ready-made examples: [Weather overlay](https://docs.mapx.at/examples/weather-overlay.md) and [Precipitation radar](https://docs.mapx.at/examples/radar-animation.md).

## Air quality and climate

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/air-quality/15.3294,48.2272?hours=6" | jq '.daily'
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/climate/15.3294,48.2272?sections=sunshine" | jq '.sunshine.hours_year'
```

Air quality follows the European Air Quality Index (1 good to 6 extremely poor), daily and hourly. Climate combines solar potential, temperature and precipitation normals, sunshine hours, heat and frost days, snow days and the drought index; choose parts with `sections`.

## Attribution

Show `mapx.at; GeoSphere Austria (CC BY 4.0)` (the `attribution` field of the response) wherever the data appear, see [Attribution and license](https://docs.mapx.at/reference/attribution.md).

## Related

- [Weather API](https://docs.mapx.at/api/weather.md), [Air quality API](https://docs.mapx.at/api/air-quality.md), [Climate API](https://docs.mapx.at/api/climate.md)
- Examples: [Weather overlay](https://docs.mapx.at/examples/weather-overlay.md), [Precipitation radar](https://docs.mapx.at/examples/radar-animation.md)

# Naming rules and units

Source: https://docs.mapx.at/reference/conventions/

> How mapx names fields and parameters: units without and with suffix, time zones (UTC except timetables), points as [lng, lat], English snake_case, errors and paging.

All mapx APIs follow the same rules. The full list of fields per endpoint is in the [API reference](https://docs.mapx.at/api/index.md).

## Names

- JSON keys, parameter names and enum values are English `snake_case` (`road_closure`, `railway_station`, `town_hall`). Only the Mapbox navigation fields in `steps` (`bannerInstructions`, `voiceInstructions`, `distanceAlongGeometry`) are camelCase.
- Ids of style layers and sources are English as well (`water-area`, `place-peak`, sources `sentinel2`, `terrain`). Sprite icon names are the `kind` of the feature (`fuel`, `peak`, `saddle`).
- Time series are called `hourly`, `daily` and, for the 15-minute nowcast, `minutely`.

## Units

| Unit | Rule | Examples |
|---|---|---|
| meters | no suffix | `distance`, `elevation`, `ascent`, `radius`, `buffer`, `snow_limit` |
| seconds | no suffix | `duration` (always a number in seconds) |
| degrees | no suffix | coordinates, `wind_direction` |
| everything else | unit as suffix | `_kmh` (km/h), `_mm`, `_hpa`, `_pct`, `_c` (°C), `_ms` (m/s), `_min`, `_wm2` |

Examples: `limit_kmh`, `precipitation_mm`, `pressure_msl_hpa`, `max_grade_pct`, `temperature_c`, `wind_speed_ms`, `sunshine_min`.

## Time zones

All timestamps are RFC 3339 in **UTC with `Z`** (measurements, forecasts, warnings, traffic reports, closures, fetch times), for example `2026-11-21T16:00:00Z`.

Exception: **timetable times of public transport** (`departure`, `arrival`, `time` of departures and the parameter `depart_at`) are Austrian local time with offset (`2026-11-22T08:15:00+01:00`). Text in MCP results also uses local time with offset.

## Points

- A point in a response is always **`[lng, lat]`** (longitude first), as in GeoJSON: `location`, `grid_location`, `geometry.coordinates`.
- A single point as query parameter is **`lng` plus `lat`**; a point written as one value is `lng,lat`, a list of points `lng,lat;lng,lat` (for example in `/v1/directions/{profile}/{coordinates}`).
- `bbox` is `west,south,east,north` (min lng, min lat, max lng, max lat).

Swapped values (`lat,lng`) are rejected with `400` `invalid-coordinates`. More in [Coordinates](https://docs.mapx.at/reference/coordinates.md).

## Responses, errors and paging

- Successful responses carry no `code`; `code` only appears in errors ([Problem Details](https://docs.mapx.at/api/errors.md), `detail` always English).
- Unknown query parameters and unknown fields in a JSON body answer `400` `unknown-parameter` with `param`.
- Paged lists return `next`: the full URL of the next page, or `null` on the last page.
- Every JSON response has an `attribution` field: `mapx.at | <source> (<license>); …` ([Attribution and license](https://docs.mapx.at/reference/attribution.md)).

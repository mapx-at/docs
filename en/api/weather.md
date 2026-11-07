# Weather

Current observations, hourly point forecast and weather tiles from open GeoSphere Austria data.

The weather API returns current observations from the nearest station, an hourly forecast for a point and ready-coloured raster tiles for temperature, precipitation, wind, humidity, cloud cover and pressure. No key, open CORS.

> All weather data comes from **GeoSphere Austria** (formerly ZAMG) under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Anyone showing the data or tiles rendered from it must credit the source, for example "Weather: GeoSphere Austria, CC BY 4.0". Every response carries a `attribution` field or the `X-Weather-Attribution` header, the TileJSON an `attribution` field.

The data is fetched centrally and cached every 10 minutes (observations) and with every new model run (forecast, about every 3 hours, 1 km resolution). Requests to the mapx API do not load the source.

## Current observations

**GET https://api.mapx.at/v1/weather/current**

<ParamTable params={[
  { name: "lon", type: "number", required: true, desc: "Longitude (WGS84), also <code>lng</code>." },
  { name: "lat", type: "number", required: true, desc: "Latitude." },
  { name: "limit", type: "integer", default: "3", desc: "Number of nearest stations, 1 to 10." },
  { name: "radius", type: "number", default: "50", desc: "Search radius in kilometres." },
]} />

Response: GeoJSON `FeatureCollection`, nearest station first (TAWES network of GeoSphere, about 280 stations). A value the station does not measure is `null`.

<ParamTable kind="response" params={[
  { name: "station_id", type: "string", desc: "Station number (Synop)." },
  { name: "name, state, altitude_m", type: "string, string, number", desc: "Name, state and altitude of the station." },
  { name: "distance_m", type: "number", desc: "Distance to the point in metres." },
  { name: "time", type: "string", desc: "Observation time, UTC." },
  { name: "temperature_c", type: "number", desc: "Air temperature in °C." },
  { name: "humidity_pct", type: "number", desc: "Relative humidity in %." },
  { name: "wind_speed_ms, wind_speed_kmh, wind_gust_ms, wind_direction_deg", type: "number", desc: "Mean wind (10 minutes), gust and the direction the wind blows from." },
  { name: "precipitation_mm_10min", type: "number", desc: "Precipitation of the last 10 minutes in mm." },
  { name: "pressure_hpa, pressure_msl_hpa", type: "number", desc: "Pressure at the station and reduced to sea level." },
]} />

<Code lang="bash" code={`# Melk
curl "https://api.mapx.at/v1/weather/current?lon=15.334&lat=48.227&limit=1"`} />

## Forecast

**GET https://api.mapx.at/v1/weather/forecast**

<ParamTable params={[
  { name: "lon", type: "number", required: true, desc: "Longitude (WGS84)." },
  { name: "lat", type: "number", required: true, desc: "Latitude." },
  { name: "hours", type: "integer", default: "24", desc: "Hours from the current hour, 1 to 60 (fewer if the model run is shorter)." },
]} />

Values come from the AROME model (`nwp-v2-1h-1km`), bilinearly interpolated to the point. Area: Austria with a margin, `404` outside.

<Code lang="json" code={`{
  "location": { "lon": 15.334, "lat": 48.227, "grid_lon": 15.3347, "grid_lat": 48.225 },
  "model": "GeoSphere AROME nwp-v2-1h-1km",
  "reference_time": "2026-11-07T12:00Z",
  "interval": "1h",
  "hours": [
    { "time": "2026-11-07T20:00Z", "temperature_c": 4.7, "precipitation_mm": 0.0, "wind_speed_ms": 4.1,
      "wind_direction_deg": 52.0, "humidity_pct": 69.9, "cloud_cover_pct": 20.0, "pressure_msl_hpa": 1027.2 }
  ],
  "attribution": "Datenquelle: GeoSphere Austria, CC BY 4.0 (https://data.hub.geosphere.at)"
}`} />

`precipitation_mm` is the total of the preceding hour. All times are UTC.

## Tiles

**GET https://api.mapx.at/v1/weather/tiles/{parameter}/{time}/{z}/{x}/{y}.png**

PNG tiles 256 × 256, semi-transparent, empty outside the forecast area. Zoom 0 to 14 (data is 1 km, beyond about zoom 12 it is only smoothly interpolated).

<ParamTable params={[
  { name: "parameter", type: "string", required: true, desc: "Quantity.", values: [
    { v: "temperature", d: "Air temperature 2 m, °C" },
    { v: "precipitation", d: "Precipitation per hour, mm/h" },
    { v: "wind", d: "Wind speed 10 m, m/s" },
    { v: "humidity", d: "Relative humidity, %" },
    { v: "cloud", d: "Cloud cover, %" },
    { v: "pressure", d: "Sea-level pressure, hPa" },
  ] },
  { name: "time", type: "string", required: true, desc: "<code>now</code> (nearest hour) or a UTC hour like <code>2026-11-07T20:00Z</code>. <code>404</code> outside the forecast." },
]} />

### TileJSON with a time dimension

**GET https://api.mapx.at/v1/weather/tiles/{parameter}.json**

The TileJSON 3.0.0 has `tiles` for `now`, `bounds`, `attribution` and the extensions:

- `mapx:time.times`: the available hours (UTC), `mapx:time.reference_time`: the model run.
- `mapx:time.template`: tile URL with a `{time}` placeholder.
- `mapx:legend`: unit and colour stops of the scale.

Tiles with a fixed hour can be cached for an hour, with `now` for five minutes.

<Tabs syncKey="lang">
  <TabItem label="JavaScript">
<Code lang="js" code={`const API = "https://api.mapx.at/v1";
const tj = await (await fetch(API + "/weather/tiles/temperature.json")).json();
const t = tj["mapx:time"].times[3];            // fourth hour
map.addSource("wetter", { type: "raster", tileSize: 256, maxzoom: 12,
  tiles: [tj["mapx:time"].template.replace("{time}", t)],
  attribution: "Weather: GeoSphere Austria, CC BY 4.0" });
map.addLayer({ id: "wetter", type: "raster", source: "wetter", paint: { "raster-opacity": 0.85 } });`} />
  </TabItem>
  <TabItem label="curl">
<Code lang="bash" code={`curl "https://api.mapx.at/v1/weather/tiles/temperature.json"
curl -o kachel.png "https://api.mapx.at/v1/weather/tiles/precipitation/now/8/137/89.png"`} />
  </TabItem>
</Tabs>

Live: [Weather overlay](/en/beispiele/wetter-overlay/).

## Errors

Fehler kommen als `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)).

| Status | Cause |
|---|---|
| `400` | `lon`/`lat` missing or not numbers, `hours`/`limit` out of range, `time` unreadable |
| `404` | point outside the forecast area, unknown parameter, time outside the forecast |
| `429` | limit exceeded, honour `Retry-After` |
| `503` | data not loaded yet (shortly after a restart) |

## Licence and origin

GeoSphere Austria Data Hub, datasets `tawes-v1-10min` (station observations) and `nwp-v2-1h-1km` (AROME). Licence [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), commercial use is allowed. The numbers are rounded or interpolated and rendered in colour, which is an adaptation of the original data. Official warnings and forecasts are published by GeoSphere Austria at [geosphere.at](https://www.geosphere.at/).
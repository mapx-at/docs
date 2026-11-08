# Air Quality API

Air quality index, pollutants and Saharan dust for a point (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Daily air quality index and hourly nitrogen dioxide, ozone, PM10 and PM2.5 near the surface plus the Saharan dust load for a point, from the GeoSphere chemical weather model (3 km). The index follows the EEA European Air Quality Index (1 good to 6 extremely poor; SO2 is not provided by the model). Map layers: the weather tiles `aqi`, `no2`, `o3`, `pm10`, `pm25` and `dust` (see [Weather API](/api/weather/)). Source: GeoSphere Austria, CC BY 4.0.

Base URL: `https://api.mapx.at` · Rate-limit zone:  · Guide: [Maps](/guides/maps/)

## Playground

<Playground group="air-quality" lang="en" />
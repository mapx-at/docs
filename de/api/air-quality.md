# Air Quality API

Luftqualitätsindex, Schadstoffe und Saharastaub für einen Punkt (GeoSphere Austria).

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Täglicher Luftqualitätsindex und stündlich Stickstoffdioxid, Ozon, PM10 und PM2.5 bodennah sowie die Saharastaub-Fracht für einen Punkt, aus dem Chemiewettermodell der GeoSphere (3 km). Der Index folgt dem Europäischen Luftqualitätsindex der EEA (1 gut bis 6 sehr schlecht; SO2 liefert das Modell nicht). Kartenebenen: die Wetterkacheln `aqi`, `no2`, `o3`, `pm10`, `pm25` und `dust` (siehe [Weather API](/de/api/weather/)). Quelle: GeoSphere Austria, CC BY 4.0.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone:  · Guide: [Karten](/de/guides/maps/)

## Playground

<Playground group="air-quality" lang="de" />
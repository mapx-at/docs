# Cycle routes tileset (GIP.at)

Source: https://docs.mapx.at/tilesets/bike-routes/

> Signed cycle routes and mountain bike trails in Austria from GIP.at, EuroVelo included, as vector tiles (tileset bike-routes) for MapLibre.

Cycle routes and mountain bike trails from the open data set of the Austrian road graph platform (GIP.at, GIPOGD), EuroVelo routes included. In the style `streets` they form the layer group `bike_routes` (off by default); see [Tourism](https://docs.mapx.at/use-cases/tourism.md).

> **Warning:** **Attribution**
> 
> Datenquelle: GIP.at, CC BY 4.0

| | |
|---|---|
| Tileset | `bike-routes`, vector, z5 to z14, as of 20260708 |
| TileJSON | `https://api.mapx.at/v1/tiles/bike-routes.json` |
| Layer | `bike_routes` (lines) with `name`, `ref`, `network`, `type`, `difficulty`, `signed`, `min_zoom` |
| Style | group `bike_routes` in `streets`, off by default |
| Query | [Tilequery](https://docs.mapx.at/api/tilequery.md) with `bike-routes` as tileset |

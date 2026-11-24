# API reference: all endpoints

Source: https://docs.mapx.at/api/

> Reference of all mapx APIs for Austria with endpoints, parameters, examples and a live playground, generated from the OpenAPI description.

All APIs live under `https://api.mapx.at/v1/` and need a free API token from [account.mapx.at](https://account.mapx.at/). Send it as `Authorization: Bearer YOUR_TOKEN` or `?access_token=YOUR_TOKEN`; see [Access and limits](https://docs.mapx.at/start/access-and-limits.md). `mx_pk_` tokens (public) work from allowed websites and Android apps with `GET`/`HEAD`, `mx_sk_` tokens (secret) from servers. The pages below are generated from the [OpenAPI description](https://api.mapx.at/v1/openapi.json) and reflect the service as of the last docs build. Each page has a playground that calls the live API.

## Conventions

- **Coordinates** are `lon,lat` (WGS84), several separated by `;`. See [Coordinates](https://docs.mapx.at/reference/coordinates.md).
- **Errors** are `application/problem+json` (RFC 9457) with a stable `code`, including `401`/`403` for token problems. See [Errors and limits](https://docs.mapx.at/api/errors.md).
- **Limits** apply per IP address and zone, plus a per-minute limit per token; `429` carries `Retry-After`. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md).
- **Caching:** tiles, styles and TileJSON carry `ETag`; send `If-None-Match` to get `304`.
- **Versioning:** the version is in the path. Fields do not change within a version, new ones are only added.

## APIs

- [Tiles](https://docs.mapx.at/api/tiles.md): Tileset list, TileJSON, XYZ tiles.
- [Styles](https://docs.mapx.at/api/styles.md): Styles, fonts, sprites, Static Tiles.
- [Geocoding](https://docs.mapx.at/api/geocoding.md): Forward, reverse and batch.
- [Search Box](https://docs.mapx.at/api/search-box.md): Suggestions, retrieve, categories.
- [Road kilometres](https://docs.mapx.at/api/road-km.md): Road kilometre to coordinates and back.
- [Address](https://docs.mapx.at/api/address.md): Autofill and validation.
- [Directions](https://docs.mapx.at/api/directions.md): Routes for car, truck, bike and foot, corridor graph.
- [Isochrone](https://docs.mapx.at/api/isochrone.md): Reachable areas by time or distance.
- [Matrix](https://docs.mapx.at/api/matrix.md): Travel times and distances between many points.
- [Map Matching](https://docs.mapx.at/api/map-matching.md): GPS traces onto the road graph.
- [Speed limits](https://docs.mapx.at/api/speed-limits.md): Statutory limits in a box or corridor.
- [Optimization](https://docs.mapx.at/api/optimization.md): Optimized trips and tour planning.
- [Public transport](https://docs.mapx.at/api/transit.md): Journeys, stops, departures.
- [Static Images](https://docs.mapx.at/api/static-images.md): Map images with overlays.
- [Tilequery](https://docs.mapx.at/api/tilequery.md): Vector tile features at a point.
- [Elevation](https://docs.mapx.at/api/elevation.md): Heights and profiles.
- [Weather](https://docs.mapx.at/api/weather.md): Observations, forecast, warnings, tiles.
- [Air quality](https://docs.mapx.at/api/air-quality.md): Index, pollutants, Saharan dust.
- [Climate](https://docs.mapx.at/api/climate.md): Solar potential, normals, snow, drought.
- [Traffic](https://docs.mapx.at/api/traffic.md): Incidents, gantry speed limits, tiles.

## More reference

- [Tilesets](https://docs.mapx.at/tilesets/index.md): data sheets of every tileset.
- [Style specification](https://docs.mapx.at/reference/style-spec.md), [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md), [Tile scheme](https://docs.mapx.at/reference/tile-scheme.md).
- [Errors and limits](https://docs.mapx.at/api/errors.md), [Attribution](https://docs.mapx.at/reference/attribution.md).
- [Status](https://api.mapx.at/health): health check of the API.

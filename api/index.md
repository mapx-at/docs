# API reference


All APIs live under `https://api.mapx.at/v1/` and need a free API token from [account.mapx.at](https://account.mapx.at/). Send it as `Authorization: Bearer YOUR_TOKEN` or `?access_token=YOUR_TOKEN`; see [Access and limits](/start/access-and-limits/). `mx_pk_` tokens (public) work from allowed websites and Android apps with `GET`/`HEAD`, `mx_sk_` tokens (secret) from servers. The pages below are generated from the [OpenAPI description](https://api.mapx.at/v1/openapi.json) and always match the running service. Each page has a playground that calls the live API.

## Conventions

- **Coordinates** are `lon,lat` (WGS84), several separated by `;`. See [Coordinates](/reference/coordinates/).
- **Errors** are `application/problem+json` (RFC 9457) with a stable `code`, including `401`/`403` for token problems. See [Errors and limits](/api/errors/).
- **Limits** are per token and zone (without a valid token per IP address); `429` carries `Retry-After`. See [Access and limits](/start/access-and-limits/).
- **Caching:** tiles, styles and TileJSON carry `ETag`; send `If-None-Match` to get `304`.
- **Versioning:** the version is in the path. Fields do not change within a version, new ones are only added. Deprecations are announced in the [Changelog](/reference/changelog/) with a sunset date.

## APIs

  - [Tiles](/api/tiles/) Tileset list, TileJSON, XYZ tiles.
  - [Styles](/api/styles/) Styles, fonts, sprites, Static Tiles.
  - [Geocoding](/api/geocoding/) Forward, reverse and batch.
  - [Search Box](/api/search-box/) Suggestions, retrieve, categories.
  - [Address](/api/address/) Autofill and validation.
  - [Directions](/api/directions/) Routes, isochrone, matrix, matching, speed limits, corridor graph.
  - [Optimization](/api/optimization/) Optimized trips and tour planning.
  - [Transit](/api/transit/) Journeys, stops, departures.
  - [Static Images](/api/static-images/) Map images with overlays.
  - [Tilequery and Parcel](/api/tilequery/) Features and parcels at a point.
  - [Elevation](/api/elevation/) Heights and profiles.
  - [Weather](/api/weather/) Observations, forecast, warnings, tiles.
  - [Air quality](/api/air-quality/) Index, pollutants, Saharan dust.
  - [Climate](/api/climate/) Solar potential, normals, snow, drought.
  - [Traffic](/api/traffic/) Incidents, gantry speed limits, tiles.
## More reference

- [Tilesets](/tilesets/): data sheets of every tileset.
- [Style specification](/reference/style-spec/), [Terrain-RGB](/reference/terrain-rgb/), [Tile scheme](/reference/tile-scheme/).
- [Errors and limits](/api/errors/), [Migration](/api/migration/), [Changelog](/reference/changelog/), [Attribution](/reference/attribution/).
- [Status](https://api.mapx.at/health): health check of the API.


# API reference

All mapx APIs: endpoints, parameters, examples and a live playground, generated from the OpenAPI description.

All APIs live under `https://api.mapx.at/v1/` and need a free API token from [account.mapx.at](https://account.mapx.at/). Send it as `Authorization: Bearer YOUR_TOKEN` or `?access_token=YOUR_TOKEN`; see [Access and limits](/start/access-and-limits/). `mx_pk_` tokens (public) work from allowed websites and Android apps with `GET`/`HEAD`, `mx_sk_` tokens (secret) from servers. The pages below are generated from the [OpenAPI description](https://api.mapx.at/v1/openapi.json) and always match the running service. Each page has a playground that calls the live API.

## Conventions

- **Coordinates** are `lon,lat` (WGS84), several separated by `;`. See [Coordinates](/reference/coordinates/).
- **Errors** are `application/problem+json` (RFC 9457) with a stable `code`, including `401`/`403` for token problems. See [Errors and limits](/api/errors/).
- **Limits** are per token and zone (without a valid token per IP address); `429` carries `Retry-After`. See [Access and limits](/start/access-and-limits/).
- **Caching:** tiles, styles and TileJSON carry `ETag`; send `If-None-Match` to get `304`.
- **Versioning:** the version is in the path. Fields do not change within a version, new ones are only added. Deprecations are announced in the [Changelog](/reference/changelog/) with a sunset date.

## APIs

                              
## More reference

- [Tilesets](/tilesets/): data sheets of every tileset.
- [Style specification](/reference/style-spec/), [Terrain-RGB](/reference/terrain-rgb/), [Tile scheme](/reference/tile-scheme/).
- [Errors and limits](/api/errors/), [Migration](/api/migration/), [Changelog](/reference/changelog/), [Attribution](/reference/attribution/).
- [Status](https://api.mapx.at/health): health check of the API.
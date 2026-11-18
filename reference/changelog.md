# Changelog: API and tileset changes

Source: https://docs.mapx.at/reference/changelog/

> Changes to the mapx APIs, tilesets and these docs, grouped as Added, Changed, Deprecated and Removed, with release dates.

## 2026-11-18

### Added
- MCP server `https://mcp.mapx.at/mcp` with 23 tools for AI assistants (Claude, ChatGPT, Cursor); its own token type AI assistant (`mx_mcp_`) that only works at the MCP server, or sign-in with the mapx account (OAuth). New errors `403` `mcp-token-only` and `mcp-token-required`. See [MCP server](https://docs.mapx.at/guides/mcp.md).
- [Terms of use](https://mapx.at/en/terms/), linked from the account, sign-up and docs.

### Changed
- Search: `proximity` now also applies to free-text search (radius query, ranking by match score, distance and importance).

### Removed
- The `mapx-gl` SDK is removed without replacement, as are the files served under `https://map.mapx.at/sdk/v1/` and the SDK pages of these docs (old addresses redirect to the examples). Instead the [examples](https://docs.mapx.at/examples/index.md) show each use case with MapLibre GL JS, OpenLayers or Leaflet and direct API calls (`fetch`).
- The former German names and duplicate fields (transition since 2026-11-08) are removed; only the English names apply. Old names return the same error as any unknown name (`404 unknown-style`, `404 unknown-tileset`, `400 invalid-parameter`); the headers `Deprecation`, `Sunset` and `Link: …; rel="successor-version"` are gone. Mapping:

| Area | old → new |
|---|---|
| Style IDs | `karte` → `streets`, `karte-morgen` → `streets-dawn`, `karte-abend` → `streets-dusk`, `karte-nacht` → `streets-night`, `karte-verkehr` → `traffic-day`, `luftbild` → `satellite`, `hybrid` → `satellite-streets`, `gelaende` → `outdoors`, `satellit-aktuell` → `satellite-current` |
| Tilesets | `basis`, `mapx-basis` → `streets`; `kataster`, `mapx-kataster` → `cadastre`; `hoehenlinien` → `contours`; `relief`, `relief-terrainrgb` → `terrain-rgb` (short name `dtm` stays); `relief-dsm-terrainrgb` → `dsm`; `dop`, `dop-ortho` → `orthophoto`; `dop_flugblock`, `dop-flugblock` → `orthophoto-flight-blocks`; `rastanlagen` → `rest-areas`; `bike_routes` → `bike-routes`; `gip_poi` → `gip-poi`; `sentinel2-datum` → `sentinel2-date` |
| Search categories | `bahnhof` → `train_station`, `haltestelle` → `bus_stop`, `kirche` → `place_of_worship_church`, `religion` → `place_of_worship`, `schule` → `school`, `universitaet` → `university`, `krankenhaus` → `hospital`, `ambulatorium` → `clinic`, `apotheke` → `pharmacy`, `arzt` → `doctor`, `rettung` → `ambulance_station`, `polizei` → `police`, `feuerwehr` → `fire_station`, `gemeindeamt` → `town_hall`, `zoll` → `customs`, `tankstelle` → `gas_station`, `ladestation` → `charging_station`, `parkplatz` → `parking`, `sharing` → `vehicle_sharing`, `rastplatz` → `rest_area`, `faehre` → `ferry_terminal`, `hubschrauberlandeplatz` → `heliport`, `mautstelle` → `toll_booth`, `huette` → `alpine_hut`, `gasthaus` → `restaurant`, `unterkunft` → `lodging`, `campingplatz` → `campground`, `gipfel` → `peak`, `pass` → `mountain_pass`, `hoehle` → `cave`, `schwimmbad` → `swimming_pool`, `sportanlage` → `sports_centre`, `golfplatz` → `golf_course`, `spielplatz` → `playground`, `tiergarten` → `zoo`, `friedhof` → `cemetery`, `bibliothek` → `library`, `theater` → `theatre`, `burg` → `castle`, `denkmal` → `monument`, `sehenswuerdigkeit` → `tourist_attraction`, `markt` → `marketplace`, `forst` → `forest`, `toilette` → `toilets`, `trinkwasser` → `drinking_water` |
| Fields `/v1/parcel` | `gstnr` → `parcel_number`, `kgnr` → `cadastral_municipality_id`, `bauflaeche` → `is_building_land`, `quelle` → `source` |
| Fields elevation and tilequery | `stichtag` → `reference_date` |
| Fields `/v1/tiles` and TileJSON | `stand` → `reference_date`, `aliases` removed; TileJSON `mapx:stand` → `mapx:reference_date` |
| Field address validation | `adrcd` → `address_code` |

## 2026-11-08

### Added
- API tokens: `api.mapx.at`, `tiles.mapx.at`, `routing.mapx.at` and `search.mapx.at` need a free token from [account.mapx.at](https://account.mapx.at/): public (`mx_pk_`, bound to origins or an Android app, `GET`/`HEAD` only) or secret (`mx_sk_`). Send it as `Authorization: Bearer` or `?access_token=`. New errors `401` (`missing-token`, `invalid-token`, `token-expired`) and `403` (`token-blocked`, `origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `ip-not-allowed`), reason in the header `X-Mapx-Reason`. Requests without a token get `401`. See [Access and limits](https://docs.mapx.at/start/access-and-limits.md) and [Errors and limits](https://docs.mapx.at/api/errors.md).
- Corridor graph: `GET /v1/corridor-graph` (`polyline` as polyline6, `buffer` 100 to 5000 m, `profile` driving, cycling or walking): compact gzip subgraph of the road network around a route (nodes, directed arcs with travel time and weight, geometry, names, road class, statutory speed limits, turn restrictions) for recomputing a route offline.
- Speed limits: `GET /v1/speedlimits` (`bbox` or `corridor` as polyline6 plus `buffer`): statutory GIP limits as compact GeoJSON.
- Directions: `exclude=ferry` (all profiles except `driving-traffic`) and `exclude=toll`; `voice_instructions` and `banner_instructions` also for `cycling`, `cycling-mtb` and `walking`; `maxspeed` carries `legal` (`false` marks a modeled speed that is not a limit).
- Climate: `GET /v1/climate/{lon},{lat}` and the section `climate` in the parcel dossier: solar potential (kWh/m² and sunshine hours per year), temperature and precipitation normals with trend, hot, frost and snow days, drought index SPEI (GeoSphere Austria, CC BY 4.0).
- Directions: `driving` and `driving-traffic` avoid binding closures (EVIS/ASFINAG) and list them in `routes[].closures_avoided`; `exclude` also for `driving-truck` and `driving-traffic`; `voice_instructions` and `banner_instructions` (Mapbox form).
- Headers `Server-Timing`, `ETag`, `Last-Modified` (styles, TileJSON, OpenAPI; conditional requests answer `304`) and `RateLimit-Policy` per endpoint group.
- Truck profile `driving-truck` with `weight`, `height`, `width`, `length` and `truck_type`.
- Error catalog with a stable `code` per error: [Errors and limits](https://docs.mapx.at/api/errors.md).
- Docs: API reference generated from OpenAPI, playground per API, guides per product.

### Changed
- English identifiers for styles, tilesets, search categories and response fields.
- Errors are `application/problem+json` with `type` URI, `code`, `param` and `allowed`; text in English, German with `Accept-Language: de`.
- The OpenAPI description is OpenAPI 3.1 with response schemas and examples.
- Directions `overview` defaults to `simplified`; `overview=full` returns every vertex.

### Deprecated (removed on 2026-11-18)
- Style IDs `karte`, `luftbild`, `gelaende`, `hybrid` and the other German IDs.
- Tileset names `basis`, `kataster`, `hoehenlinien`, `relief`, `dop`, `dop_flugblock`.
- The 49 German search category IDs (for example `bahnhof`, `apotheke`).
- Parcel fields `gstnr`, `kgnr`, `bauflaeche`, `quelle` and elevation field `stichtag`.

### Removed
- The alias paths `/v1/{tileset}.json` and `/v1/{tileset}/{z}/{x}/{y}.{ext}`. Use `/v1/tiles/…`.

## 2026-11-07

### Added
- Public transport: journeys (`/v1/directions/transit`), departures (`/v1/departures`) and stops (`/v1/stops`).
- Tolls in Directions: `exclude=toll`, `routes[].toll`, `toll_road`, `toll_road_m`.
- Traffic: `/v1/traffic/*` (incidents, speed limits, signs, tiles), `incidents` in Directions, profile `driving-traffic`.
- Tilesets `rastanlagen` (rest areas) and `sharing` (shared mobility), search categories for both.
- Weather: observations, hourly forecast and weather tiles (GeoSphere Austria, CC BY 4.0).
- Static Images: aerial images, terrain and map styles as PNG or WebP with overlays.
- Tilequery, Parcel, Search Box, Address Autofill and Address Validation, batch geocoding.
- Directions: `alternatives`, `annotations`, `radiuses`; `matrix`, `isochrone`, `matching`, optimized trips and tour planning.
- SDK mapx-gl.

### Changed
- Reverse geocoding answers in milliseconds instead of about a second.
- XYZ tiles moved under `/v1/tiles`.
- Attribution in responses; `operationId` on every endpoint in the OpenAPI.

## Earlier

### Added
- Tilesets DTM, DSM, contours, base map, cadastre, orthophoto and flight blocks as PMTiles, TileJSON and XYZ.
- Elevation and profile API.
- Documentation in German and English.

## Tileset releases

| Tileset | Source date | File |
|---|---|---|
| terrain-rgb | 20250915 | `relief-terrainrgb-20250915-ed2e4341.pmtiles` |
| dsm | 20250915 | `relief-dsm-terrainrgb-20250915-d19cb434.pmtiles` |

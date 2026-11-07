# Changelog

Changes to data products, APIs and these docs.

## 08.11.2026

- Changed: error responses (`application/problem+json`) now carry a stable `code`, a `type` URI into the new [error catalog](/en/api/errors/) (instead of `about:blank`) and, where useful, `param` and `allowed`. `title` and `detail` are English, German with `Accept-Language: de`. Unparsable path coordinates (e.g. `/v1/parcel/abc`) now answer `400` instead of `404`.
- Changed: the OpenAPI description is OpenAPI 3.1, fully English, with response schemas and examples for the main endpoints, a server list, rate-limit description and `/v1/geocode/{q}`.
- Changed: `overview` for Directions defaults to `simplified` (as in Mapbox, much smaller responses on long routes); `overview=full` returns every vertex as before.
- New: headers `Server-Timing` (`router`, `search`, `render`, `total`), `ETag` and `Last-Modified` for styles, TileJSON and OpenAPI (conditional requests answer `304`), `RateLimit-Policy` per endpoint group.
- Removed: the former alias paths `/v1/{tileset}.json` and `/v1/{tileset}/{z}/{x}/{y}.{ext}` (use `/v1/tiles/…`) and `/v1/directions/driving-ev/…`. Both were already blocked at the gateway (`404`).

- Deprecated: the German API identifiers are replaced by English ones. The old names keep working as aliases **until 2027-05-31** (sunset) and return the headers `Deprecation`, `Sunset` and `Link: …; rel="successor-version"`. All mappings: [Migration to English names](/en/api/migration/).
  - Style IDs: `karte` → `streets`, `karte-morgen` → `streets-dawn`, `karte-abend` → `streets-dusk`, `karte-nacht` → `streets-night`, `karte-verkehr` → `traffic-day`, `luftbild` → `satellite`, `hybrid` → `satellite-streets`, `gelaende` → `outdoors`.
  - Tilesets: `basis` → `streets`, `kataster` → `cadastre`, `hoehenlinien` → `contours`, `relief` → `terrain-rgb`, `dop` → `orthophoto`, `dop_flugblock` → `orthophoto-flight-blocks` (`dtm`, `dsm` stay).
  - Search categories: 49 new snake_case IDs, for example `bahnhof` → `train_station`, `apotheke` → `pharmacy`, `schule` → `school`.
  - Fields: parcel `gstnr`/`kgnr`/`bauflaeche`/`quelle` → `parcel_number`/`cadastral_municipality_id`/`is_building_land`/`source`, elevation `stichtag` → `reference_date` (both fields are in the response until the sunset).
  - Style metadata: `mapx:stil` → `mapx:style`, `mapx:api-stil` → `mapx:api_style`.

## 07.11.2026

- New: Sentinel-2 satellite imagery for Austria (best-pixel composite, updated daily, z5 to z13) as tileset `sentinel2`.
- New: Public transport with current and following-year timetables, stops merged across feeds (IFOPT), departure monitor `/v1/departures` and stops `/v1/stops`.
- New: Tolls in Directions (`exclude=toll`, `routes[].toll`, `toll_road`, `toll_road_m`).
- New: Tilesets `rastanlagen` (ASFINAG rest areas) and `sharing` (shared-mobility locations), search categories `rastplatz` and `sharing`.
- New: Traffic messages `/v1/traffic/*` (incidents, variable message signs) and `incidents[]` in Directions.
- Changed: Reverse geocoding answers in 3 to 60 ms instead of about 1 s; XYZ tiles under `/v1/tiles`; attribution in responses; `operationId` on every endpoint in the OpenAPI.

- New: [Weather](/en/api/weather/) (`/v1/weather/current`, `/v1/weather/forecast`, weather tiles with TileJSON and a time dimension) from open GeoSphere Austria data, CC BY 4.0; example [Weather overlay](/en/beispiele/wetter-overlay/).

- New: [Static Images](/en/api/static/) (`/v1/static`, aerial imagery and terrain as PNG or WebP with overlays), [SDK mapx-gl](/en/api/sdk/); English docs brought up to date with the German ones. Also: [Examples](/en/beispiele/) with twelve live examples, and API pages for [Styles](/en/api/styles/), [Geocoding](/en/api/geocoding/) and [Routing](/en/api/routing/).

- New: data products DTM, DSM, contour lines, base map, cadastre, orthophoto and flight blocks, each with a live map and layer table.
- New: API with tiles (PMTiles and XYZ), TileJSON, elevation and profile, and a test console.
- New: documentation in German and English.

## Data releases

<table>
  <thead><tr><th>Product</th><th>Reference date</th><th>File</th><th>Created</th></tr></thead>
  <tbody>
    {Object.values(data.tilesets).filter((x) => ["relief", "dsm"].includes(x.key)).map((x) => (
      <tr><td>{x.tileset}</td><td>{x.stand}</td><td><code>{x.datei}</code></td><td>{x.erzeugt ?? ""}</td></tr>
    ))}
  </tbody>
</table>
# Changelog

Changes to the APIs, tilesets and these docs, grouped as Added, Changed, Deprecated and Removed.

Deprecations carry a sunset date: from that day the old name is removed. Details of the rename: [Migration](/api/migration/).

## 2026-11-08

### Added
- Climate: `GET /v1/climate/{lon},{lat}` and the section `climate` in the parcel dossier: solar potential (kWh/m² and sunshine hours per year), temperature and precipitation normals with trend, hot, frost and snow days, drought index SPEI (GeoSphere Austria, CC BY 4.0).
- Directions: `driving` and `driving-traffic` avoid binding closures (EVIS/ASFINAG) and list them in `routes[].closures_avoided`; `exclude` also for `driving-truck` and `driving-traffic`; `voice_instructions` and `banner_instructions` (Mapbox form).
- Headers `Server-Timing`, `ETag`, `Last-Modified` (styles, TileJSON, OpenAPI; conditional requests answer `304`) and `RateLimit-Policy` per endpoint group.
- Truck profile `driving-truck` with `weight`, `height`, `width`, `length` and `truck_type`.
- Error catalog with a stable `code` per error: [Errors and limits](/api/errors/).
- Docs: API reference generated from OpenAPI, playground per API, guides per product.

### Changed
- English identifiers for styles, tilesets, search categories and response fields.
- Errors are `application/problem+json` with `type` URI, `code`, `param` and `allowed`; text in English, German with `Accept-Language: de`.
- The OpenAPI description is OpenAPI 3.1 with response schemas and examples.
- Directions `overview` defaults to `simplified`; `overview=full` returns every vertex.

### Deprecated (sunset 2027-05-31)
- Style IDs `karte`, `luftbild`, `gelaende`, `hybrid` and the other German IDs.
- Tileset names `basis`, `kataster`, `hoehenlinien`, `relief`, `dop`, `dop_flugblock`.
- The 49 German search category IDs (for example `bahnhof`, `apotheke`).
- Parcel fields `gstnr`, `kgnr`, `bauflaeche`, `quelle` and elevation field `stichtag` (both names are in the response until the sunset).

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

<table>
  <thead><tr><th>Tileset</th><th>Source date</th><th>File</th></tr></thead>
  <tbody>
    {Object.values(data.tilesets).filter((x) => ["relief", "dsm"].includes(x.key)).map((x) => (
      <tr><td>{x.key === "relief" ? "terrain-rgb" : x.key}</td><td>{x.stand}</td><td><code>{x.datei}</code></td></tr>
    ))}
  </tbody>
</table>
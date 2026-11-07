# Changelog

Changes to data products, APIs and these docs.

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
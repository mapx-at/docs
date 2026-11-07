# Elevation and profile

Elevation for points or along a path from DTM or DSM, GeoJSON response, with test console.

Elevation for one point, several points or a path. Machine-readable: [openapi.yaml](/openapi.yaml), live [/v1/openapi.json](https://api.mapx.at/v1/openapi.json).

<Endpoint url="https://api.mapx.at/v1/elevation" lang="en" />

<ParamTable lang="en" params={[
  { name: "points", type: "string", desc: "<code>lon,lat;lon,lat;...</code> (WGS84, longitude before latitude)" },
  { name: "path", type: "string", desc: "<code>lon,lat;lon,lat;...</code>, polyline (2 to 512 vertices); together with <code>samples</code> a profile along the path" },
  { name: "samples", type: "integer", desc: "Number of sample points evenly spaced by length (great circle on the sphere) along <code>path</code>, 2 to 1024, default 100" },
  { name: "model", type: "string", desc: "<code>dtm</code> (default) or <code>dsm</code>" },
]} />

`points` and `path` are mutually exclusive; `points` accepts at most 512 points. Elevation is interpolated bilinearly from the z16 tiles (0.1 m grid).

### Try it

The console sends a real request to the service.

<Testkonsole lang="en" />

### Example: point

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation?points=12.694,47.074&model=dtm"`} />

Response: GeoJSON `FeatureCollection`. Each feature is a point (coordinates `[lon, lat]`) with the properties `elevation` (m), `resolution_m` (grid spacing of the source, 1 m), `model` and `stichtag` (reference date); with `path` also `distance_m` (distance from the start of the path). Outside the data, `elevation` is `null` (the response stays `200`).

<Code lang="json" code={`{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "geometry": { "type": "Point", "coordinates": [12.694, 47.074] },
    "properties": { "elevation": 0.0, "resolution_m": 1.0, "model": "dtm", "stichtag": "2025-09-15" }
  }]
}`} />

### Example: profile

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50"`} />

Response as above, one feature per sample point in path order.

## Errors

`application/problem+json` with `type`, `title`, `status` and `detail`:

<Code lang="json" code={`{
  "type": "about:blank",
  "title": "Bad Request",
  "status": 400,
  "detail": "'12.5': expected lon,lat"
}`} />

Typical cases: `400` (parameters missing or malformed), `404` (tileset or tile unknown), `429` (too many requests, with `Retry-After`), `503` (service temporarily unavailable).
`type` is `about:blank`; `title` is the status text, `detail` the explanation.

More: [Errors, limits and operations](/en/api/fehler/).
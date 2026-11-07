# Elevation and profile

Elevation for points or along a path from the DTM or DSM, GeoJSON response.

The elevation API returns terrain height ([DTM](/en/daten/dtm/)) or surface height ([DSM](/en/daten/dsm/)) for single points or as a profile along a line. No key, open CORS.

**Use cases:** elevation profile of a hiking or cycling route, height of a site (construction, radio, solar), building and tree height as DSM minus DTM, checking GPS altitudes.

## Retrieve elevation

<Endpoint url="https://api.mapx.at/v1/elevation" />

### Parameters

Exactly one of `points` or `path` is required.

<ParamTable params={[
  { name: "points", type: "string", desc: "Points as <code>lon,lat;lon,lat;…</code> (WGS84, longitude first), up to 512." },
  { name: "path", type: "string", desc: "Line as <code>lon,lat;lon,lat;…</code>, 2 to 512 vertices. The response is a profile with <code>samples</code> points." },
  { name: "samples", type: "integer", default: "100", desc: "Only with <code>path</code>: number of points spaced evenly by distance, 2 to 1024." },
  { name: "model", type: "string", default: "dtm", desc: "Elevation model.", values: [
    { v: "dtm", d: "Bare ground without buildings and vegetation" },
    { v: "dsm", d: "Surface incl. buildings, trees, bridges" },
  ] },
]} />

Heights are bilinearly interpolated from the 1 m grid, resolution 0.1 m.

### Try it

<Testkonsole />

### Example requests

<Tabs syncKey="lang">
  <TabItem label="curl">
<Code lang="bash" code={`# Grossglockner and Melk, terrain
curl "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.334,48.227"

# Profile with 50 points, surface
curl "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50&model=dsm"`} />
  </TabItem>
  <TabItem label="JavaScript">
<Code lang="js" code={`const r = await fetch("https://api.mapx.at/v1/elevation?points=12.694,47.074");
const { features } = await r.json();
console.log(features[0].properties.elevation); // 3726.3`} />
  </TabItem>
  <TabItem label="Python">
<Code lang="python" code={`import requests
r = requests.get("https://api.mapx.at/v1/elevation",
                 params={"path": "12.69,47.07;12.70,47.08", "samples": 50})
for f in r.json()["features"]:
    print(f["properties"]["distance_m"], f["properties"]["elevation"])`} />
  </TabItem>
</Tabs>

## Response

A GeoJSON <code>FeatureCollection</code>, one <code>Point</code> feature per point in request order (for <code>path</code>, along the line).

<ParamTable kind="response" params={[
  { name: "elevation", type: "number | null", desc: "Height in metres, one decimal (height reference of the source; for the DTM EVRF2000 orthometric). <code>null</code> outside the data area." },
  { name: "distance_m", type: "number", desc: "Only for <code>path</code>: distance from the start of the line in metres." },
  { name: "model", type: "string", desc: "Model used", values: [{ v: "dtm", d: "DTM" }, { v: "dsm", d: "DSM" }] },
  { name: "resolution_m", type: "number", desc: "Grid spacing of the source in metres (1)." },
  { name: "stichtag", type: "string", desc: "Reference date of the model, ISO date." },
]} />

<Code lang="json" code={`{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "geometry": { "type": "Point", "coordinates": [12.694, 47.074] },
    "properties": { "elevation": 3726.3, "resolution_m": 1.0, "model": "dtm", "stichtag": "2025-09-15" }
  }]
}`} />

## Errors

Errors are returned as `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)).

| Status | Cause |
|---|---|
| `400` | neither or both of <code>points</code>/<code>path</code>, coordinate not <code>lon,lat</code>, too many points, invalid <code>samples</code> or <code>model</code> |
| `429` | rate limit exceeded, honour <code>Retry-After</code> |
| `503` | temporarily unavailable |

<Code lang="json" code={`{ "type": "about:blank", "title": "Bad Request", "status": 400, "detail": "model: dtm oder dsm" }`} />

## Limits

- Up to 512 points per request (<code>points</code> or vertices of <code>path</code>), up to 1024 profile samples.
- Rate limit per IP address; large requests count more. Details: [Errors and limits](/en/api/fehler/).
- Machine-readable: [openapi.json](https://api.mapx.at/v1/openapi.json).
# Routing

Directions, isochrones, matrix and map matching on the GIP.at network for walking, cycling and driving.

The basis is the Austrian transport network [GIP.at](https://www.gip.at) (CC BY 4.0). The endpoints follow the Mapbox shape, `/v1/...` with the profile in the path, and need no access token. Profiles: `walking`, `cycling`, `driving` (a `mapbox/` prefix is accepted). Coordinates appear in the path as `lon,lat`, several points separated by `;`.

## Directions

**GET https://api.mapx.at/v1/directions/{profile}/{coordinates}**

Equivalent: `GET /v1/directions?profile=…&coordinates=…`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Mode of transport", values: [{ v: "walking", d: "on foot" }, { v: "cycling", d: "bicycle" }, { v: "driving", d: "car" }] },
  { name: "coordinates", type: "string", required: true, desc: "Start, optional intermediate points and destination as <code>lon,lat;lon,lat;…</code>." },
  { name: "geometries", type: "string", desc: "Geometry format, <code>geojson</code> returns a GeoJSON line." },
  { name: "overview", type: "string", desc: "Level of detail of the geometry, for example <code>full</code>." },
  { name: "steps", type: "boolean", default: "false", desc: "With <code>true</code>, turn instructions in <code>legs[].steps</code>." },
  { name: "language", type: "string", desc: "Language of the instructions, for example <code>de</code>." },
]} />

Response: `code` (`Ok`) and `routes[]` with `distance` (metres), `duration` (seconds), `legs` and `geometry`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/cycling/15.33,48.23;15.34,48.24?geometries=geojson&overview=full"`} />

Live: [Route](/en/beispiele/route/).

## Isochrone

**GET https://api.mapx.at/v1/isochrone/{profile}/{coordinate}**

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "<code>walking</code>, <code>cycling</code> or <code>driving</code>." },
  { name: "coordinate", type: "string", required: true, desc: "Start point <code>lon,lat</code>." },
  { name: "contours_minutes", type: "string", default: "5,10,15", desc: "Time steps in minutes, comma-separated, up to four values." },
  { name: "contours_meters", type: "string", desc: "Alternatively, distance steps in metres." },
]} />

Response: `FeatureCollection`, one `MultiPolygon` per step (largest first) with `contour`, `unit`, `metric` and `area_km2`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/isochrone/walking/15.33,48.23?contours_minutes=5,10,15"`} />

Live: [Reachability](/en/beispiele/isochrone/).

## Matrix

**GET https://api.mapx.at/v1/matrix/{profile}/{coordinates}**

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "As above." },
  { name: "coordinates", type: "string", required: true, desc: "All places as <code>lon,lat;lon,lat;…</code>." },
  { name: "sources", type: "string", desc: "Indices of the sources, separated by <code>;</code>; all if omitted." },
  { name: "destinations", type: "string", desc: "Indices of the destinations, separated by <code>;</code>; all if omitted." },
  { name: "annotations", type: "string", desc: "<code>duration</code>, <code>distance</code> or both, comma-separated." },
]} />

Response: `durations` (seconds) and, on request, `distances` (metres) as a matrix; `null` means not routable; plus `sources` and `destinations` with the snapped point.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/matrix/driving/15.33,48.23;15.34,48.24;15.3,48.2"`} />

Live: [Travel-time matrix](/en/beispiele/matrix/).

## Map Matching

**GET https://api.mapx.at/v1/matching/{profile}/{coordinates}**

Snaps a GPS trace to the road network.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "As above." },
  { name: "coordinates", type: "string", required: true, desc: "Trace as <code>lon,lat;lon,lat;…</code>." },
  { name: "timestamps", type: "string", desc: "Seconds per point, separated by <code>;</code>; improves the matching." },
  { name: "radiuses", type: "string", desc: "Search radius per point in metres, separated by <code>;</code>." },
  { name: "geometries", type: "string", desc: "Geometry format, <code>geojson</code> for GeoJSON." },
]} />

Response: `matchings[]` with `geometry`, `distance`, `duration` and `confidence`, plus `tracepoints` (snapped point or `null`).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/matching/driving/15.33019,48.2299;15.3295,48.229?geometries=geojson"`} />

## Errors and limits

Errors are returned as `application/problem+json` with a matching HTTP status (`400` invalid input, `404` no route or no network nearby, `429` rate limit). Isochrone and matrix have tight limits. General: [Errors and limits](/en/api/fehler/).
# Routing

Directions, isochrones, matrix and map matching on the GIP.at network for walking, cycling and driving.

The basis is the Austrian transport network [GIP.at](https://www.gip.at) (CC BY 4.0). The endpoints follow the Mapbox shape, `/v1/...` with the profile in the path, and need no access token. Profiles: `walking`, `cycling`, `driving`, `driving-traffic` (car with current traffic and closures, see [Traffic](/en/api/traffic/)) (a `mapbox/` prefix is accepted). Coordinates appear in the path as `lon,lat`, several points separated by `;`.

## Directions

**GET https://api.mapx.at/v1/directions/{profile}/{coordinates}**

Equivalent: `GET /v1/directions?profile=…&coordinates=…`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Mode of transport", values: [{ v: "walking", d: "on foot" }, { v: "cycling", d: "bicycle" }, { v: "driving", d: "car" }, { v: "driving-traffic", d: "car with live traffic and closures" }] },
  { name: "coordinates", type: "string", required: true, desc: "Start, optional intermediate points and destination as <code>lon,lat;lon,lat;…</code>." },
  { name: "geometries", type: "string", desc: "Geometry format, <code>geojson</code> returns a GeoJSON line." },
  { name: "overview", type: "string", desc: "Level of detail of the geometry, for example <code>full</code>." },
  { name: "steps", type: "boolean", default: "false", desc: "With <code>true</code>, turn instructions in <code>legs[].steps</code>." },
  { name: "language", type: "string", desc: "Language of the instructions, for example <code>de</code>." },
  { name: "alternatives", type: "boolean | integer", default: "false", desc: "<code>true</code> returns up to two more routes in <code>routes[]</code>, a number from 0 to 3 limits the count. Only with exactly two coordinates." },
  { name: "annotations", type: "string", desc: "<code>duration</code>, <code>distance</code>, <code>speed</code>, comma-separated. Returns <code>legs[].annotation</code> with one value per geometry segment (seconds, metres, metres per second)." },
  { name: "incidents", type: "boolean", default: "true for driving-traffic", desc: "Traffic reports along the route in <code>legs[].incidents</code> (car only), see <a href=\"/en/api/traffic/\">Traffic</a>." },
  { name: "radiuses", type: "string", desc: "Search radius for snapping to the network per coordinate in metres, separated by <code>;</code>, or <code>unlimited</code>." },
  { name: "approaches", type: "string", desc: "<code>unrestricted</code> or <code>curb</code> per coordinate. Validated, but not yet used by the router." },
  { name: "bearings", type: "string", desc: "<code>degrees,tolerance</code> per coordinate, separated by <code>;</code> (empty leaves a point free). Validated, but not yet used by the router." },
]} />

Response: `code` (`Ok`) and `routes[]` with `distance` (metres), `duration` (seconds), `legs` and `geometry`.

**Alternatives** come from laterally offset via points: the router computes start → via point → destination for ten points along the fastest route and keeps paths that take at most 1.4 times as long, contain no U-turn and share at most 70 percent with the main route and with each other. `routes[0]` is always the fastest route, the others follow by duration. The request costs several times a plain route against the rate limit.

**Exclusions:** `exclude=motorway` and `exclude=toll` (also comma separated, profile `driving` only). `motorway` blocks motorways (GIP category A); expressways stay allowed. `toll` blocks the special-toll sections (A9 Gleinalm and Bosruck, A10 Tauern and Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg); vignette roads stay allowed, use `motorway` for those. Other values give `400`. If start or destination lies on a blocked section the router finds no route.

**Tolls in the response:** Every car route (`driving`, also `driving-ev`) contains `routes[].toll`: `vignette_required` (at least 300 m on vignette roads), `vignette_m` (metres on vignette roads), `vignette_products[]` (car vignettes with `name` and `amount` in euro), `special_toll[]` (special-toll stations on the way with `name`, `amount`, `location`), `special_toll_total` and `currency`. A toll station counts once per trip even if the data lists several directional sections. Source: ASFINAG, toll information, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The figures apply to cars without trailer; prices are those of the data set for 2026. **Toll links:** Every route also carries `routes[].toll_road` (`true` if it runs over links with the GIP attribute TOLL, i.e. special-toll sections such as A13 Brenner) and `toll_road_m` (metres on them). With `steps=true` each step has `intersections[0].classes` in Mapbox style: `toll` (step touches special-toll links), `motorway`, `ferry`. Vignette roads are not `toll`; they appear in `toll.vignette_*`. The special-toll price is in `toll.special_toll[]`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/cycling/15.33,48.23;15.34,48.24?geometries=geojson&overview=full"`} />

Live: [Route](/en/beispiele/route/).

## EV routing

> Until the terms of use of the E-Control charging data are settled the endpoint answers `404`.


**GET https://api.mapx.at/v1/directions/driving-ev/{coordinates}**

Exactly start and destination. The car route is computed; if the range is too short the service picks charging stations along the route ([data](/en/daten/ladestationen/)) and routes via them. Linear range model: constant consumption per kilometre, start with a full battery, charge up to `ev_charge_to`, never arrive below `ev_reserve`.

<ParamTable params={[
  { name: "ev_range_km", type: "number", desc: "Required. Range on a full battery in km (1 to 1500)." },
  { name: "ev_charge_kw_min", type: "number", default: "50", desc: "Minimum charging power of stations in kW." },
  { name: "ev_battery_kwh", type: "number", default: "60", desc: "Battery size, used for charging time." },
  { name: "ev_charge_kw_max", type: "number", default: "100", desc: "Highest charging power of the vehicle in kW." },
  { name: "ev_charge_to", type: "number", default: "0.8", desc: "Charge target as share of the battery." },
  { name: "ev_reserve", type: "number", default: "0.1", desc: "Reserve on arrival as share of the battery." },
  { name: "steps, language", type: "", desc: "as for Directions." },
]} />

The response is a Directions response, each entry of `legs[]` ends at a charging station. `routes[0].ev` holds `stops[]` (`id`, `name`, `location`, `max_kw`, `along_m`, `offset_m`, `soc_arrival`, `soc_departure`, `charge_s`), `drive_s`, `charge_s` and `total_s`. The route `duration` is driving time only, `ev.total_s` includes charging. If a stage has no station the service answers `400` and names the section. Stations more than 3 km from the route are not used. Data source: E-Control charging station directory.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/driving-ev/16.37,48.21;11.40,47.27?ev_range_km=300&ev_charge_kw_min=100"`} />

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

## Optimized trips

**GET https://api.mapx.at/v1/optimized-trips/{profile}/{coordinates}**

Finds the best order for 2 to 25 points (travelling salesman, Mapbox Optimization API v1) and returns the route in that order. Up to 12 points are solved exactly, beyond that with nearest neighbour, 2-opt and Or-opt on the travel-time matrix.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "As above." },
  { name: "coordinates", type: "string", required: true, desc: "2 to 25 places as <code>lon,lat;lon,lat;…</code>." },
  { name: "source", type: "string", default: "any", desc: "<code>first</code>: the first point is the start. <code>any</code>: any start." },
  { name: "destination", type: "string", default: "any", desc: "<code>last</code>: the last point is the end. <code>any</code>: any end." },
  { name: "roundtrip", type: "boolean", default: "true", desc: "With <code>true</code> the trip returns to the start. With <code>destination=last</code>, <code>source=first</code> is required; the path from the first to the last point is searched and the way back to the start is added." },
  { name: "steps", type: "boolean", default: "false", desc: "Turn instructions in <code>legs[].steps</code>." },
  { name: "geometries", type: "string", default: "geojson", desc: "<code>geojson</code>, <code>polyline</code> or <code>polyline6</code>." },
  { name: "overview", type: "string", desc: "<code>false</code> omits the geometry." },
  { name: "language", type: "string", default: "de", desc: "<code>de</code> or <code>en</code>." },
]} />

Response: `trips[]` with one route (`distance`, `duration`, `legs`, `geometry`) and `waypoints[]` in input order. Each entry has `waypoint_index` (position in the trip) and `trips_index` (always 0). If points are not connected, the answer is `422`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/optimized-trips/driving/15.3338,48.2279;15.2065,48.2111;15.0864,48.1696;15.3969,48.2339?source=first&roundtrip=true"`} />

Like matrix and isochrone, the request counts as expensive.

For tours with several vehicles, capacities and time windows see [Vehicle routing](/en/api/optimization/).

## Errors and limits

Errors are returned as `application/problem+json` with a matching HTTP status (`400` invalid input, `404` no route or no network nearby, `429` rate limit). Isochrone and matrix have tight limits. General: [Errors and limits](/en/api/fehler/).
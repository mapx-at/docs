# Traffic

Incident reports (accidents, roadworks, closures), traffic regulations, variable message signs and routing around closures from official Austrian sources.

The traffic API returns incident reports (accidents, roadworks, closures with time span) and traffic regulations for all of Austria, the state of the ASFINAG variable message signs on motorways and expressways as GeoJSON or vector tiles, and the routing profile `driving-traffic`, which avoids closures. No key, open CORS.

> Sources: **EVIS.AT** by ASFINAG (planned and unplanned events, traffic regulations from states, cities, police and ASFINAG; CC BY 4.0 after registration) and the ASFINAG **variable message signs** (CC BY 4.0). The road network comes from [GIP.at](https://www.gip.at) (CC BY 4.0). Credit: "Traffic data: ASFINAG / EVIS.AT". Every response carries an `attribution` field.
> 
> Reports with `mandatory: true` (EVIS `compliance-option` mandatory) and all traffic regulations are binding: the router treats them as closures, and maps must display them unchanged (do not shorten or reinterpret `text`). The paid EVIS data for area-wide traffic state, forecast and daily profiles is not used; there are therefore no congestion levels.

EVIS is fetched centrally by the routing service (checksum every 2 minutes, data only on change), the variable message signs every 2 minutes. Requests to the mapx API never hit the sources.

## Incident reports

**GET https://api.mapx.at/v1/traffic/incidents**

<ParamTable params={[
  { name: "bbox", type: "string", desc: "<code>west,south,east,north</code> in degrees. Without <code>bbox</code> all reports are returned, including those without a location." },
  { name: "planned", type: "boolean", desc: "<code>true</code> planned only, <code>false</code> unplanned only." },
  { name: "road", type: "string", desc: "Road number, for example <code>A12</code>." },
  { name: "active", type: "boolean", default: "true", desc: "<code>false</code> also returns ended and future reports (default once <code>from</code> or <code>to</code> is set)." },
  { name: "type", type: "string", desc: "Comma-separated: <code>road_closure</code>, <code>construction</code>, <code>lane_restriction</code>, <code>accident</code>, <code>congestion</code>, <code>road_hazard</code>, <code>miscellaneous</code>." },
  { name: "from, to", type: "string", desc: "Time window in UTC, <code>YYYY-MM-DD</code> or <code>YYYY-MM-DDTHH:MMZ</code>: reports whose time span touches the window." },
  { name: "route", type: "string", desc: "Only reports along a route: encoded polyline (as <code>geometries=polyline6</code>) or a GeoJSON <code>LineString</code>. Travel direction is respected; sorted by position on the route." },
  { name: "route_precision", type: "number", default: "6", desc: "Polyline precision, 5 or 6." },
  { name: "buffer", type: "number", default: "25", desc: "Corridor half-width in metres (1 to 1000)." },
  { name: "limit, offset", type: "number", default: "200, 0", desc: "Paging, <code>limit</code> at most 1000. The response gives <code>total</code> and, if more follow, <code>next_offset</code>." },
]} />

Long routes do not fit in a URL: the same parameters can be sent as a JSON object via `POST` (there `route` may be a GeoJSON object).

Response: GeoJSON `FeatureCollection` with the geometry along the GIP links (`LineString` or `MultiLineString`).

<ParamTable kind="response" params={[
  { name: "type, impact", type: "string", desc: "Type in Mapbox vocabulary (see <code>type</code>) and severity: <code>critical</code> (closed), <code>major</code> (binding, lanes, accident), <code>minor</code>, <code>low</code> (planned). Derived by mapx from kind and closure flag." },
  { name: "geometry_index_start, geometry_index_end", type: "number", desc: "Only with <code>route</code>: affected vertices of the given route." },
  { name: "kind, category", type: "string", desc: "Type (German), for example <code>Straßensperre</code>, <code>Bauarbeiten</code>, <code>Unfall</code>, <code>Verkehrssteuerungsmaßnahme</code>." },
  { name: "mandatory", type: "boolean", desc: "Binding report: counts as a closure in routing." },
  { name: "both_directions", type: "boolean", desc: "Applies in both travel directions." },
  { name: "text", type: "string", desc: "ASFINAG report text (German)." },
  { name: "road", type: "string", desc: "Road number." },
  { name: "planned, closed", type: "boolean", desc: "Planned event, road fully closed." },
  { name: "start, end, updated", type: "string", desc: "Time span and last change, UTC." },
]} />

<Code lang="bash" code={`# Reports on the A12 (Inn valley)
curl "https://api.mapx.at/v1/traffic/incidents?road=A12"

# Closures and road works around Linz in December
curl "https://api.mapx.at/v1/traffic/incidents?bbox=14.1,48.2,14.5,48.4&type=road_closure,construction&from=2026-12-01&to=2026-12-31"

# Reports along a route (geometry from /v1/directions with geometries=polyline6)
curl -X POST https://api.mapx.at/v1/traffic/incidents -H "Content-Type: application/json" \\
  -d '{"route": {"type": "LineString", "coordinates": [[16.37, 48.21], [16.30, 48.19]]}, "buffer": 30}'`} />

## Speed limits (variable message signs)

**GET https://api.mapx.at/v1/traffic/flow**

<ParamTable params={[
  { name: "bbox", type: "string", required: true, desc: "<code>west,south,east,north</code> in degrees." },
]} />

Motorway and expressway links where a variable message sign with a position shows a speed limit below free flow (`speed_kmh`, `free_kmh`, `level` 0 to 3 by ratio). In the router the limit applies to the link at the sign, not to the whole stretch up to the next sign.

## Vector tiles and style

**GET https://api.mapx.at/v1/traffic/tiles.json**

The tiles `/v1/traffic/tiles/{z}/{x}/{y}.pbf` (zoom 5 to 14, 60 seconds cache) contain the layers `traffic` (`level`, `speed`, `free`) and `incidents` (`kind`, `text`, `road`, `planned`, `closed`). The `karte*` styles include the `verkehr-*` layers hidden (`visibility: none`); the style `karte-verkehr` shows them. To switch on: `map.setLayoutProperty("verkehr-lage", "visibility", "visible")` for every layer prefixed `verkehr-`. In the viewer on mapx.at, "Layers → Traffic" switches them on (off by default).

## Routing around closures

The profile `driving-traffic` works for `directions`, `matrix` and `isochrone`. Binding closures (EVIS `mandatory`, traffic regulations for cars) are blocked. Non-binding road closures and time-conditioned regulations (weekdays and hours only appear in the text) are strongly avoided but stay passable so the network stays connected; the travel time is unchanged. Speed limits from the message signs lengthen the travel time. There is no area-wide traffic state: all other links keep the standard weights. It routes without contraction hierarchies and is slower than `driving`; without a data snapshot it answers `503`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/driving-traffic/10.74,47.24;10.66,47.07"`} />

**Incidents on the route:** With `driving-traffic` (and with `incidents=true` also with `driving`) every leg contains `routes[].legs[].incidents` in Mapbox shape: `id`, `type`, `description` (report text unchanged), `sub_type` (kind from the source), `geometry_index_start` and `geometry_index_end` (vertices of the leg), `closed`, `impact`, `mandatory`, `start_time`, `end_time`, `creation_time`, `affected_road_names`. Active reports in the travel direction within 25 m of the route are included. `incidents=false` turns it off. Attribution in `incidents_attribution`.

## Variable message signs

**GET https://api.mapx.at/v1/traffic/signs**

Current state of the ASFINAG variable message signs: `speed_kmh` (displayed speed limit), `pictogram`, `text` (supplementary text), `image` (ASFINAG image), `working`. The source gives a position for only a small share of the units (about 40 of 5300); the rest refer to ASFINAG's unit table and only show up in `total_signs`. Optional `bbox`.

## Status

**GET https://api.mapx.at/v1/traffic/status**

Shows data age (`fetched`), report and section counts, and the links closed or slowed in the router.
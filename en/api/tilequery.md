# Tilequery and parcel

Vector features around a point with distance (like Mapbox Tilequery), plus parcel and elevation at a point.

Tilequery answers "what is at this point?" without fetching and decoding tiles yourself. The server reads the vector tile at the tileset's highest zoom level (and the neighbouring tiles for a radius), computes the distance in metres and returns Mapbox-style GeoJSON. No key, CORS open.

## Tilequery

**GET https://api.mapx.at/v1/tilequery/{tileset}/{lon},{lat}.json**

`tileset` is a name from the [tile list](/en/api/kacheln/) (for example `streets`, `cadastre`, `contours`; the former names `basis`, `kataster`, `hoehenlinien` stay valid as aliases until 2027-05-31, see [Migration](/en/api/migration/)). Several tilesets are comma-separated (at most 15): `streets,cadastre`. All names via `GET https://api.mapx.at/v1/tiles` ([tileset list](/en/api/tiles/#tileset-list)).

<ParamTable params={[
  { name: "radius", type: "number", default: "0", desc: "Radius in metres, 0 to 2000. With 0 only features containing the point (polygons) are returned." },
  { name: "limit", type: "integer", default: "5", desc: "Number of results, 1 to 50." },
  { name: "layers", type: "string", desc: "Comma-separated layer names, for example <code>building_a,road_l</code>." },
  { name: "geometry", type: "string", desc: "One geometry type only", values: [{ v: "point", d: "points" }, { v: "linestring", d: "lines" }, { v: "polygon", d: "areas" }] },
  { name: "dedupe", type: "boolean", default: "true", desc: "Remove duplicates caused by tile borders (same layer and id, otherwise same properties)." },
]} />

The response is a `FeatureCollection`, nearest feature first. Each feature carries its tile attributes plus `properties.tilequery`:

<ParamTable kind="response" params={[
  { name: "tilequery.distance", type: "number", desc: "Distance to the point in metres (0.1 m). Polygons containing the point have 0." },
  { name: "tilequery.geometry", type: "string", desc: "<code>point</code>, <code>linestring</code> or <code>polygon</code>." },
  { name: "tilequery.layer", type: "string", desc: "Layer name." },
  { name: "tilequery.tileset", type: "string", desc: "Only with several tilesets." },
]} />

Geometries are clipped to the tile edge as in the tile. With `dtm` or `dsm` as tileset the elevation comes back as a point feature (`properties.elevation`).

<Code lang="bash" code={`# Buildings and paths within 30 m of the Melk abbey square
curl "https://api.mapx.at/v1/tilequery/streets/15.3338,48.2279.json?radius=30&layers=building_a,road_l"`} />

## Parcel at a point

**GET https://api.mapx.at/v1/parcel/{lon},{lat}.json**

Returns the parcel containing the point as a polygon with `parcel_number` (parcel number; building parcels start with a dot, for example `.14`), `cadastral_municipality_id` (cadastral community number), `area_m2` (area in m²), `is_building_land` and `source`. Source is the BEV cadastre (DKM) in the `cadastre` tile layer. The former fields `gstnr`, `kgnr`, `bauflaeche` and `quelle` are deprecated and still included until 2027-05-31 ([Migration](/en/api/migration/)).

- The cadastre tiles only contain boundary lines. Polygon and area are computed from them; the area differs from the register value by parts per thousand.
- Parcels reaching beyond the tile window around the point (about 2 km) are missing.
- Without cadastre data at the point (outside the built regions, road space without a parcel) the list is empty.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/parcel/15.3338,48.2279.json"`} />

## Elevation at a point

**GET https://api.mapx.at/v1/elevation/{lon},{lat}.json**

One `Feature` with `properties.dtm` (terrain), `properties.dsm` (surface), `resolution_m` and `reference_date` (formerly `stichtag`, deprecated until 2027-05-31). Several points and profiles: [Elevation](/en/api/hoehe/).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation/15.3338,48.2279.json"`} />

## Errors

`400` for an invalid coordinate, radius or geometry type, `404` for an unknown tileset, `429` over the limit (tilequery counts twice, parcel four times). See [Errors and limits](/en/api/fehler/).
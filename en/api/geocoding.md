# Search and geocoding

Search places, addresses, peaks and parcels (forward) and resolve coordinates to addresses (reverse).

## Forward search

**GET https://api.mapx.at/v1/geocode**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Search text: place, address, peak, parcel (for example <code>Melk .111/1</code>)." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: prefer hits near this point." },
  { name: "types", type: "string", desc: "Comma-separated: <code>address</code>, <code>street</code>, <code>locality</code>, <code>place</code>, <code>district</code>, <code>region</code>, <code>postcode</code>, <code>poi</code>." },
  { name: "bbox", type: "string", desc: "<code>minLon,minLat,maxLon,maxLat</code>: restrict hits to an area." },
  { name: "limit", type: "integer", default: "5", desc: "Number of hits, 1 to 10." },
]} />

The response is a Mapbox-style GeoJSON `FeatureCollection`. Each feature has `text`, `place_name` (with context, for example "Krummnußbaum, Bezirk Melk, Niederösterreich"), `place_type`, `relevance`, `center`, `geometry` and usually a `bbox` for zooming; `properties` contains `mapx_type`, `context` and a zoom suggestion.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/geocode?q=Krummnußbaum&limit=2"`} />

Live: [Search](/en/beispiele/suche/).

## Reverse search

**GET https://api.mapx.at/v1/reverse**

<ParamTable params={[
  { name: "lng, lat", type: "number", required: true, desc: "Point in WGS84." },
  { name: "radius", type: "number", desc: "Search radius in metres, 1 to 5000, default 500." },
  { name: "limit", type: "integer", desc: "Number of hits, 1 to 10, default 1." },
  { name: "types", type: "string", desc: "As for <code>/v1/geocode</code>." },
]} />

Returns the nearest address (or other matching places) as a `FeatureCollection` in the same shape as the forward search, nearest hit first. Without `types`, parcels and POIs are not searched. Typical response time below 50 ms.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/reverse?lng=15.33&lat=48.23"`} />

## Batch geocoding

**POST https://api.mapx.at/v1/geocode/batch**

Up to 1000 requests per call. The body is a JSON array (`Content-Type: application/json`, at most 512 KB); the response is an array of `FeatureCollection`s in the same order. One call counts as ten simple requests against the rate limit (plus one point per 32 KB of body). Each request:

<ParamTable params={[
  { name: "q", type: "string", desc: "Search text. Either <code>q</code> or the structured fields." },
  { name: "street, housenumber, postcode, place", type: "string", desc: "Structured address (<code>street</code> is required). With <code>housenumber</code> and no <code>types</code> only addresses are searched." },
  { name: "types", type: "string | array", desc: "As for <code>/v1/geocode</code>, also as a list." },
  { name: "limit", type: "integer", default: "5", desc: "1 bis 10." },
  { name: "proximity", type: "array", desc: "<code>[lon, lat]</code> (or text <code>lon,lat</code>)." },
  { name: "bbox", type: "array", desc: "<code>[minLon, minLat, maxLon, maxLat]</code>." },
]} />

One invalid entry rejects the whole call with `400`; the error message names the entry number.

<Code lang="bash" code={`curl -X POST https://api.mapx.at/v1/geocode/batch -H 'Content-Type: application/json' \\
  -d '[{"q":"Hauptplatz Melk","limit":1},{"street":"Hauptplatz","housenumber":"3","postcode":"3390"}]'`} />
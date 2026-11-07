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
  { name: "radius", type: "number", desc: "Search radius in metres." },
  { name: "limit", type: "integer", desc: "Number of hits." },
  { name: "types", type: "string", desc: "As for <code>/v1/geocode</code>." },
]} />

Returns the nearest address (or other matching places) as a `FeatureCollection` in the same shape as the forward search.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/reverse?lng=15.33&lat=48.23"`} />
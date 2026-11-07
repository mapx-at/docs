# Address autofill

Complete addresses as you type: street, house number, postcode and place from the official address register.

Suggestions come only from the official address register (BEV, CC BY 4.0), no places, streets or parcels. Two steps as with Mapbox: <code>suggest</code> while typing, <code>retrieve</code> after the user picks one.

## Suggestions

**GET https://api.mapx.at/v1/autofill/suggest**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Partial input, for example <code>Hauptpl 3 Mel</code>." },
  { name: "session_token", type: "string", desc: "Freely chosen session id (1 to 64 letters, digits, <code>-</code>, <code>_</code>), echoed in the response." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: prefer nearby addresses." },
  { name: "limit", type: "integer", default: "5", desc: "1 to 10." },
]} />

The response contains <code>suggestions</code>. Each suggestion carries the fields separately:

<ParamTable params={[
  { name: "id", type: "string", desc: "Address id (<code>address-&lt;AdrCD&gt;</code>) for <code>retrieve</code>." },
  { name: "address_line1", type: "string", desc: "Street and house number." },
  { name: "street, housenumber, postcode, place", type: "string", desc: "Street, house number, postcode, delivery place." },
  { name: "municipality, state", type: "string", desc: "Municipality and state." },
  { name: "full_address", type: "string", desc: "One line for display." },
  { name: "coordinates", type: "array", desc: "<code>[lon, lat]</code>" },
]} />

<Code lang="bash" code={`curl "https://api.mapx.at/v1/autofill/suggest?q=Hauptpl%203%20Mel&session_token=abc-123&limit=3"`} />

## Retrieve an address

**GET https://api.mapx.at/v1/autofill/retrieve/{id}**

Returns the chosen address as a <code>FeatureCollection</code> with one point feature; <code>properties</code> has the same fields as a suggestion. Unknown id: <code>404</code>.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/autofill/retrieve/address-6444189?session_token=abc-123"`} />
# Address validation

Check an address against the official address register, correct it and add coordinates and address id.

Checks an address against the official address register (BEV, CC BY 4.0). It recognises spelling variants such as <code>Str.</code> and <code>Straße</code>, abbreviations (<code>G.</code>, <code>Pl.</code>), umlauts and <code>ß</code> (<code>Müllerweg</code>, <code>Mueller-Weg</code>) as well as typos.

**GET https://api.mapx.at/v1/address/validate**
**POST https://api.mapx.at/v1/address/validate**

<ParamTable params={[
  { name: "q", type: "string", desc: "Free text, for example <code>Hauptstr. 12a, 3390 Melk</code>. Either <code>q</code> or the fields." },
  { name: "street", type: "string", desc: "Street (required for structured input)." },
  { name: "housenumber, postcode, place", type: "string", desc: "House number, postcode, place (optional)." },
]} />

With <code>GET</code> as query parameters, with <code>POST</code> as a JSON object (at most 16 KB).

## Response

<ParamTable params={[
  { name: "status", type: "string", desc: "<code>match</code>: correct (a missing postcode/place is only added). <code>corrected</code>: unique, but with deviations. <code>ambiguous</code>: several equally good addresses, see <code>candidates</code>. <code>not_found</code>: nothing suitable." },
  { name: "confidence", type: "number", desc: "0 to 1." },
  { name: "address", type: "object", desc: "Normalised address: <code>address_line1</code>, <code>street</code>, <code>housenumber</code>, <code>postcode</code>, <code>place</code>, <code>municipality</code>, <code>state</code>, <code>full_address</code>. <code>null</code> for <code>ambiguous</code> and <code>not_found</code>." },
  { name: "deviations", type: "array", desc: "Per field <code>field</code>, <code>input</code>, <code>expected</code> and <code>kind</code>: <code>spelling</code>, <code>wrong</code> (wrong value, for example postcode), <code>added</code> (was missing, filled in)." },
  { name: "coordinates", type: "array", desc: "<code>[lon, lat]</code>" },
  { name: "id, adrcd", type: "string", desc: "Address id (<code>address-&lt;AdrCD&gt;</code>) and the register's AdrCD." },
  { name: "candidates", type: "array", desc: "For <code>ambiguous</code> up to 5 addresses in the shape of <code>address</code>." },
]} />

<Code lang="bash" code={`curl -G "https://api.mapx.at/v1/address/validate" --data-urlencode "q=Hauptpl. 3, 3391 Melk"`} />
<Code lang="json" code={`{"status": "corrected", "confidence": 0.85,
 "address": {"address_line1": "Hauptplatz 3", "street": "Hauptplatz", "housenumber": "3", "postcode": "3390", "place": "Melk", "municipality": "Melk", "state": "Niederösterreich"},
 "deviations": [{"field": "street", "input": "Hauptpl.", "expected": "Hauptplatz", "kind": "spelling"},
                {"field": "postcode", "input": "3391", "expected": "3390", "kind": "wrong"}],
 "coordinates": [15.329874, 48.227329], "id": "address-6444189", "adrcd": "6444189"}`} />
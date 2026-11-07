# Adressvalidierung

Eine Adresse gegen das amtliche Adressregister prüfen, korrigieren und mit Koordinate und Adress-ID versehen.

Prüft eine Adresse gegen das amtliche Adressregister (BEV, CC BY 4.0). Erkannt werden Schreibvarianten wie <code>Str.</code> und <code>Straße</code>, Abkürzungen (<code>G.</code>, <code>Pl.</code>), Umlaute und <code>ß</code> (<code>Müllerweg</code>, <code>Mueller-Weg</code>) sowie Tippfehler.

**GET https://api.mapx.at/v1/address/validate**
**POST https://api.mapx.at/v1/address/validate**

<ParamTable params={[
  { name: "q", type: "string", desc: "Freitext, zum Beispiel <code>Hauptstr. 12a, 3390 Melk</code>. Entweder <code>q</code> oder die Felder." },
  { name: "street", type: "string", desc: "Straße (Pflicht bei strukturierter Eingabe)." },
  { name: "housenumber, postcode, place", type: "string", desc: "Hausnummer, PLZ, Ort (optional)." },
]} />

Per <code>GET</code> als Abfrageparameter, per <code>POST</code> als JSON-Objekt (höchstens 16 KB).

## Antwort

<ParamTable params={[
  { name: "status", type: "string", desc: "<code>match</code>: stimmt (fehlende PLZ/Ort werden nur ergänzt). <code>corrected</code>: eindeutig, aber mit Abweichungen. <code>ambiguous</code>: mehrere gleichwertige Adressen, siehe <code>candidates</code>. <code>not_found</code>: nichts Passendes." },
  { name: "confidence", type: "number", desc: "0 bis 1." },
  { name: "address", type: "object", desc: "Normalisierte Adresse: <code>address_line1</code>, <code>street</code>, <code>housenumber</code>, <code>postcode</code>, <code>place</code>, <code>municipality</code>, <code>state</code>, <code>full_address</code>. <code>null</code> bei <code>ambiguous</code> und <code>not_found</code>." },
  { name: "deviations", type: "array", desc: "Je Feld <code>field</code>, <code>input</code>, <code>expected</code> und <code>kind</code>: <code>spelling</code> (Schreibweise), <code>wrong</code> (falscher Wert, zum Beispiel PLZ), <code>added</code> (fehlte, wurde ergänzt)." },
  { name: "coordinates", type: "array", desc: "<code>[lon, lat]</code>" },
  { name: "id, adrcd", type: "string", desc: "Adress-Id (<code>address-&lt;AdrCD&gt;</code>) und AdrCD des Adressregisters." },
  { name: "candidates", type: "array", desc: "Bei <code>ambiguous</code> bis zu 5 Adressen wie in <code>address</code>." },
]} />

<Code lang="bash" code={`curl -G "https://api.mapx.at/v1/address/validate" --data-urlencode "q=Hauptpl. 3, 3391 Melk"`} />
<Code lang="json" code={`{"status": "corrected", "confidence": 0.85,
 "address": {"address_line1": "Hauptplatz 3", "street": "Hauptplatz", "housenumber": "3", "postcode": "3390", "place": "Melk", "municipality": "Melk", "state": "Niederösterreich"},
 "deviations": [{"field": "street", "input": "Hauptpl.", "expected": "Hauptplatz", "kind": "spelling"},
                {"field": "postcode", "input": "3391", "expected": "3390", "kind": "wrong"}],
 "coordinates": [15.329874, 48.227329], "id": "address-6444189", "adrcd": "6444189"}`} />
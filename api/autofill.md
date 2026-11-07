# Adress-Autofill

Adressen beim Tippen vervollständigen: Straße, Hausnummer, PLZ und Ort aus dem amtlichen Adressregister.

Vorschläge nur aus dem amtlichen Adressregister (BEV, CC BY 4.0), keine Orte, Straßen oder Grundstücke. Zwei Schritte wie bei Mapbox: <code>suggest</code> beim Tippen, <code>retrieve</code> nach der Auswahl.

## Vorschläge

**GET https://api.mapx.at/v1/autofill/suggest**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Teileingabe, zum Beispiel <code>Hauptpl 3 Mel</code>." },
  { name: "session_token", type: "string", desc: "Frei gewählte Sitzungskennung (1 bis 64 Zeichen aus Buchstaben, Ziffern, <code>-</code>, <code>_</code>), wird in der Antwort zurückgegeben." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: nahe Adressen bevorzugen." },
  { name: "limit", type: "integer", default: "5", desc: "1 bis 10." },
]} />

Die Antwort enthält <code>suggestions</code>. Jeder Vorschlag trägt die Felder getrennt:

<ParamTable params={[
  { name: "id", type: "string", desc: "Adress-Id (<code>address-&lt;AdrCD&gt;</code>) für <code>retrieve</code>." },
  { name: "address_line1", type: "string", desc: "Straße und Hausnummer." },
  { name: "street, housenumber, postcode, place", type: "string", desc: "Straße, Hausnummer, PLZ, Zustellort." },
  { name: "municipality, state", type: "string", desc: "Gemeinde und Bundesland." },
  { name: "full_address", type: "string", desc: "Eine Zeile für die Anzeige." },
  { name: "coordinates", type: "array", desc: "<code>[lon, lat]</code>" },
]} />

<Code lang="bash" code={`curl "https://api.mapx.at/v1/autofill/suggest?q=Hauptpl%203%20Mel&session_token=abc-123&limit=3"`} />

## Adresse abrufen

**GET https://api.mapx.at/v1/autofill/retrieve/{id}**

Liefert die gewählte Adresse als <code>FeatureCollection</code> mit einem Punkt-Feature; <code>properties</code> hat dieselben Felder wie ein Vorschlag. Unbekannte Id: <code>404</code>.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/autofill/retrieve/address-6444189?session_token=abc-123"`} />
# Suche und Geocoding

Orte, Adressen, Gipfel und Grundstücke suchen (Vorwärts) und Koordinaten in Adressen auflösen (Rückwärts).

> `/v1/geocode` und `/v1/reverse` werden gerade ausgerollt. Bis dahin steht der Suchindex direkt zur Verfügung (siehe unten). Anfragen von `docs.mapx.at` und eigenen Seiten brauchen eine CORS-Freigabe der Herkunft.

## Vorwärtssuche

**GET https://api.mapx.at/v1/geocode**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Suchtext: Ort, Adresse, Gipfel, Grundstück (zum Beispiel <code>Melk .111/1</code>)." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: Treffer in der Nähe bevorzugen." },
  { name: "types", type: "string", desc: "Kommagetrennt, Werte wie <code>place</code>, <code>address</code>, <code>street</code>, <code>peak</code>, <code>parcel</code>." },
  { name: "limit", type: "integer", default: "5", desc: "Anzahl Treffer, höchstens 10." },
]} />

Die Antwort ist eine GeoJSON-`FeatureCollection` mit Punkt-Features (`properties.label`, `context`, `type`) und `bbox` zum Zoomen.

## Rückwärtssuche

**GET https://api.mapx.at/v1/reverse**

<ParamTable params={[
  { name: "lon, lat", type: "number", required: true, desc: "Punkt in WGS84." },
  { name: "types", type: "string", desc: "Wie bei <code>/v1/geocode</code>." },
]} />

Liefert die nächstgelegene Adresse, Gemeinde und Katastralgemeinde als `FeatureCollection`.

## Suchindex direkt (verfügbar)

**POST https://search.mapx.at/indexes/places/search**

Meilisearch mit schreibgeschütztem Search-only-Key im Header `Authorization: Bearer …` (der Key steht im [Live-Beispiel](/beispiele/suche/)). Filter zum Beispiel `type = parcel` oder `type != parcel`; `limit` und `q` wie bei Meilisearch.

<Code lang="bash" code={`curl -X POST https://search.mapx.at/indexes/places/search \\
  -H "Authorization: Bearer <SEARCH_KEY>" -H "Content-Type: application/json" \\
  -d '{"q":"Melk Bahnhof","limit":3}'`} />

Live: [Suche](/beispiele/suche/).
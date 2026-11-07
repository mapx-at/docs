# Suche und Geocoding

Orte, Adressen, Gipfel und Grundstücke suchen (Vorwärts) und Koordinaten in Adressen auflösen (Rückwärts).

## Vorwärtssuche

**GET https://api.mapx.at/v1/geocode**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Suchtext: Ort, Adresse, Gipfel, Grundstück (zum Beispiel <code>Melk .111/1</code>)." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: Treffer in der Nähe bevorzugen." },
  { name: "types", type: "string", desc: "Kommagetrennt: <code>address</code>, <code>street</code>, <code>locality</code>, <code>place</code>, <code>district</code>, <code>region</code>, <code>postcode</code>, <code>poi</code> (Orte, Bahnhöfe/Haltestellen, Grundstücke)." },
  { name: "bbox", type: "string", desc: "<code>minLon,minLat,maxLon,maxLat</code>: Treffer auf ein Gebiet beschränken." },
  { name: "limit", type: "integer", default: "5", desc: "Anzahl Treffer, 1 bis 10." },
]} />

Die Antwort ist eine GeoJSON-`FeatureCollection` im Mapbox-Stil. Jedes Feature hat `text`, `place_name` (mit Kontext, zum Beispiel „Krummnußbaum, Bezirk Melk, Niederösterreich“), `place_type`, `relevance`, `center`, `geometry` und meist eine `bbox` zum Zoomen; `properties` enthält `mapx_type`, `context` und einen Zoom-Vorschlag.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/geocode?q=Krummnußbaum&limit=2"`} />

Live: [Suche](/beispiele/suche/).

## Rückwärtssuche

**GET https://api.mapx.at/v1/reverse**

<ParamTable params={[
  { name: "lng, lat", type: "number", required: true, desc: "Punkt in WGS84." },
  { name: "radius", type: "number", desc: "Suchradius in Metern, 1 bis 5000, Standard 500." },
  { name: "limit", type: "integer", desc: "Anzahl Treffer, 1 bis 10, Standard 1." },
  { name: "types", type: "string", desc: "Wie bei <code>/v1/geocode</code>." },
]} />

Liefert die nächstgelegene Adresse (oder andere passende Orte) als `FeatureCollection` in derselben Form wie die Vorwärtssuche, nächster Treffer zuerst. Ohne `types` werden Grundstücke und POIs nicht gesucht. Typische Antwortzeit unter 50 ms.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/reverse?lng=15.33&lat=48.23"`} />

## Batch-Geocoding

**POST https://api.mapx.at/v1/geocode/batch**

Bis zu 1000 Anfragen in einem Aufruf. Der Rumpf ist ein JSON-Array (`Content-Type: application/json`, höchstens 512 KB), die Antwort ein Array von `FeatureCollection`s in derselben Reihenfolge. Ein Aufruf zählt im Rate-Limit wie zehn einfache Anfragen (plus ein Punkt je 32 KB Rumpf). Jede Anfrage:

<ParamTable params={[
  { name: "q", type: "string", desc: "Suchtext. Entweder <code>q</code> oder die strukturierten Felder." },
  { name: "street, housenumber, postcode, place", type: "string", desc: "Strukturierte Adresse (<code>street</code> ist Pflicht). Mit <code>housenumber</code> werden ohne <code>types</code> nur Adressen gesucht." },
  { name: "types", type: "string | array", desc: "Wie bei <code>/v1/geocode</code>, auch als Liste." },
  { name: "limit", type: "integer", default: "5", desc: "1 bis 10." },
  { name: "proximity", type: "array", desc: "<code>[lon, lat]</code> (auch Text <code>lon,lat</code>)." },
  { name: "bbox", type: "array", desc: "<code>[minLon, minLat, maxLon, maxLat]</code>." },
]} />

Ein ungültiger Eintrag lehnt den ganzen Aufruf mit `400` ab; die Fehlermeldung nennt die Nummer des Eintrags.

<Code lang="bash" code={`curl -X POST https://api.mapx.at/v1/geocode/batch -H 'Content-Type: application/json' \\
  -d '[{"q":"Hauptplatz Melk","limit":1},{"street":"Hauptplatz","housenumber":"3","postcode":"3390"}]'`} />
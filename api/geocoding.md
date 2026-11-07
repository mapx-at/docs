# Suche und Geocoding

Orte, Adressen, Gipfel und Grundstücke suchen (Vorwärts) und Koordinaten in Adressen auflösen (Rückwärts).

## Vorwärtssuche

**GET https://api.mapx.at/v1/geocode**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Suchtext: Ort, Adresse, Gipfel, Grundstück (zum Beispiel <code>Melk .111/1</code>)." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: Treffer in der Nähe bevorzugen." },
  { name: "types", type: "string", desc: "Kommagetrennt: <code>address</code>, <code>street</code>, <code>locality</code>, <code>place</code>, <code>district</code>, <code>region</code>, <code>postcode</code>, <code>poi</code>." },
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
  { name: "radius", type: "number", desc: "Suchradius in Metern." },
  { name: "limit", type: "integer", desc: "Anzahl Treffer." },
  { name: "types", type: "string", desc: "Wie bei <code>/v1/geocode</code>." },
]} />

Liefert die nächstgelegene Adresse (oder andere passende Orte) als `FeatureCollection` in derselben Form wie die Vorwärtssuche.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/reverse?lng=15.33&lat=48.23"`} />
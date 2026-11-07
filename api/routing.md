# Routing

Directions, Isochrone, Matrix und Map Matching auf dem GIP.at-Netz für Fuß, Rad und Auto.

> Die vereinheitlichten Pfade `/v1/directions`, `/v1/isochrone`, `/v1/matrix` und `/v1/matching` werden gerade ausgerollt. Der Routing-Dienst selbst läuft unter `https://routing.mapx.at` mit `/route`, `/isochrone`, `/matrix` und `/match`; die Parameter unten gelten für beide Pfade. Anfragen von `docs.mapx.at` und eigenen Seiten brauchen eine CORS-Freigabe der Herkunft.

Grundlage ist das österreichische Verkehrsnetz [GIP.at](https://www.gip.at) (CC BY 4.0). Profile: `foot`, `bike`, `car`.

## Directions

**GET https://api.mapx.at/v1/directions**

Aktuell: `GET https://routing.mapx.at/route`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "Verkehrsmittel", values: [{ v: "foot", d: "zu Fuß" }, { v: "bike", d: "Rad" }, { v: "car", d: "Auto" }] },
  { name: "from", type: "string", required: true, desc: "Start als <code>lon,lat</code>." },
  { name: "to", type: "string", required: true, desc: "Ziel als <code>lon,lat</code>." },
  { name: "steps", type: "boolean", default: "false", desc: "Mit <code>true</code> Abbiegehinweise in <code>properties.steps</code>." },
  { name: "lang", type: "string", default: "de", desc: "Sprache der Hinweise, <code>de</code> oder <code>en</code>." },
  { name: "radius", type: "number", desc: "Suchradius in Metern, um Start und Ziel ans Netz zu binden." },
]} />

Antwort: GeoJSON-`Feature` mit `LineString` und den Eigenschaften `distance_m`, `duration_s`, `from`/`to` (gebundener Punkt mit Abstand) sowie optional `steps`.

<Code lang="bash" code={`curl "https://routing.mapx.at/route?profile=bike&from=15.33,48.23&to=15.34,48.24"`} />

Live: [Route](/beispiele/route/).

## Isochrone

**GET https://api.mapx.at/v1/isochrone**

Aktuell: `GET` oder `POST https://routing.mapx.at/isochrone`.

<ParamTable params={[
  { name: "profile", type: "string", required: true, desc: "<code>foot</code>, <code>bike</code> oder <code>car</code>." },
  { name: "location", type: "string", required: true, desc: "Startpunkt <code>lon,lat</code> (bei POST ein Array <code>[lon,lat]</code>)." },
  { name: "contours", type: "string", desc: "Fahr- oder Gehzeiten in Minuten, kommagetrennt, höchstens 10 Stufen und 120 Minuten. Entweder <code>contours</code> oder <code>distances</code>." },
  { name: "distances", type: "string", desc: "Distanzen in Metern, kommagetrennt, bis 150 km." },
  { name: "simplify, buffer, resolution", type: "number", desc: "Glättung der Ränder in Metern, optional." },
]} />

Antwort: `FeatureCollection`, je Stufe ein `MultiPolygon` (größte zuerst) mit `contour`, `unit`, `metric` und `area_km2`.

<Code lang="bash" code={`curl "https://routing.mapx.at/isochrone?profile=foot&location=15.33,48.23&contours=5,10,15"`} />

Live: [Erreichbarkeit](/beispiele/isochrone/).

## Matrix

**POST https://api.mapx.at/v1/matrix**

Aktuell: `POST https://routing.mapx.at/matrix`, JSON-Body.

<ParamTable params={[
  { name: "profile", type: "string", desc: "Wie oben, Standard <code>car</code>." },
  { name: "sources", type: "number[][]", required: true, desc: "Liste von <code>[lon,lat]</code>, höchstens 1000." },
  { name: "destinations", type: "number[][]", desc: "Ziele; ohne Angabe gelten die Quellen auch als Ziele. Höchstens 250.000 Paare." },
]} />

Antwort: `durations` (Sekunden) und `distances` (Meter) als Matrix, `null` heißt nicht routbar; dazu `sources` und `destinations` mit gebundenem Punkt.

Live: [Fahrzeitmatrix](/beispiele/matrix/).

## Map Matching

**POST https://api.mapx.at/v1/matching**

Aktuell: `POST https://routing.mapx.at/match`, JSON-Body. Ordnet eine GPS-Spur dem Wegenetz zu.

<ParamTable params={[
  { name: "coordinates", type: "number[][]", required: true, desc: "Spur als <code>[lon,lat]</code>, mindestens 2, höchstens 10.000 Punkte." },
  { name: "profile", type: "string", desc: "Wie oben." },
  { name: "timestamps", type: "number[]", desc: "Sekunden je Punkt, verbessert die Zuordnung." },
  { name: "radius", type: "number", desc: "Suchradius je Punkt in Metern, höchstens 500." },
  { name: "sigma", type: "number", desc: "Messrauschen der Spur in Metern, höchstens 100." },
]} />

Antwort: `FeatureCollection` mit je einem `LineString` je zusammenhängendem Teilstück und `tracepoints` (gebundener Punkt mit Konfidenz, oder `null`).

## Fehler und Limits

Fehler kommen als Text oder JSON mit passendem HTTP-Status (`400` ungültige Eingabe, `404`/`422` keine Route, `429` Limit). Die Rechenlast begrenzt der Dienst je IP; Isochrone und Matrix haben enge Limits. Allgemeines: [Fehler und Limits](/api/fehler/).
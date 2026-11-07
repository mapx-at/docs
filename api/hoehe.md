# Höhe und Profil

Höhe für Punkte oder entlang eines Pfads aus DTM oder DSM, GeoJSON-Antwort.

Die Höhen-API liefert die Geländehöhe ([DTM](/daten/dtm/)) oder die Oberflächenhöhe ([DSM](/daten/dsm/)) für einzelne Punkte oder als Profil entlang einer Linie. Kein Schlüssel, CORS offen.

**Nutzungen:** Höhenprofil einer Wander- oder Radroute, Höhe eines Standorts (Bauplanung, Funk, Solar), Gebäude- und Baumhöhe als DSM minus DTM, Plausibilisierung von GPS-Höhen.

## Höhe abfragen

**GET https://api.mapx.at/v1/elevation**

### Parameter

Genau einer von `points` oder `path` ist Pflicht.

<ParamTable params={[
  { name: "points", type: "string", desc: "Punkte als <code>lon,lat;lon,lat;…</code> (WGS84, Länge vor Breite), höchstens 512." },
  { name: "path", type: "string", desc: "Linie als <code>lon,lat;lon,lat;…</code>, 2 bis 512 Stützpunkte. Die Antwort ist ein Profil mit <code>samples</code> Messpunkten." },
  { name: "samples", type: "integer", default: "100", desc: "Nur mit <code>path</code>: Anzahl gleichmäßig nach Strecke verteilter Messpunkte, 2 bis 1024." },
  { name: "model", type: "string", default: "dtm", desc: "Höhenmodell.", values: [
    { v: "dtm", d: "Gelände ohne Gebäude und Bewuchs" },
    { v: "dsm", d: "Oberfläche mit Gebäuden, Bäumen, Brücken" },
  ] },
]} />

Die Höhe wird bilinear aus dem 1-m-Raster interpoliert, Auflösung 0,1 m.

### Ausprobieren

<Testkonsole />

### Beispielanfragen

<Tabs syncKey="lang">
  <TabItem label="curl">
<Code lang="bash" code={`# Großglockner und Melk, Gelände
curl "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.334,48.227"

# Profil mit 50 Punkten, Oberfläche
curl "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50&model=dsm"`} />
  </TabItem>
  <TabItem label="JavaScript">
<Code lang="js" code={`const r = await fetch("https://api.mapx.at/v1/elevation?points=12.694,47.074");
const { features } = await r.json();
console.log(features[0].properties.elevation); // 3726.3`} />
  </TabItem>
  <TabItem label="Python">
<Code lang="python" code={`import requests
r = requests.get("https://api.mapx.at/v1/elevation",
                 params={"path": "12.69,47.07;12.70,47.08", "samples": 50})
for f in r.json()["features"]:
    print(f["properties"]["distance_m"], f["properties"]["elevation"])`} />
  </TabItem>
</Tabs>

## Antwort

GeoJSON <code>FeatureCollection</code>, ein <code>Point</code>-Feature je Punkt in Anfragereihenfolge (bei <code>path</code> entlang der Linie).

<ParamTable kind="response" params={[
  { name: "elevation", type: "number | null", desc: "Höhe in Metern, eine Nachkommastelle (Höhenbezug der Quelle, beim DTM EVRF2000 orthometrisch). <code>null</code> außerhalb des Datengebiets." },
  { name: "distance_m", type: "number", desc: "Nur bei <code>path</code>: Strecke vom Linienanfang in Metern." },
  { name: "model", type: "string", desc: "Verwendetes Modell", values: [{ v: "dtm", d: "DTM" }, { v: "dsm", d: "DSM" }] },
  { name: "resolution_m", type: "number", desc: "Rasterweite der Quelle in Metern (1)." },
  { name: "stichtag", type: "string", desc: "Stand des Höhenmodells, ISO-Datum." },
]} />

<Code lang="json" code={`{
  "type": "FeatureCollection",
  "features": [{
    "type": "Feature",
    "geometry": { "type": "Point", "coordinates": [12.694, 47.074] },
    "properties": { "elevation": 3726.3, "resolution_m": 1.0, "model": "dtm", "stichtag": "2025-09-15" }
  }]
}`} />

## Profil-Endpunkt

**GET https://api.mapx.at/v1/elevation/profile**

<ParamTable params={[
  { name: "coordinates", type: "string", required: true, desc: "Linie als <code>lon,lat;lon,lat;…</code>, 2 bis 512 Stützpunkte." },
  { name: "samples", type: "integer", default: "100", desc: "Anzahl gleichmäßig nach Strecke verteilter Messpunkte, 2 bis 1024." },
  { name: "model", type: "string", default: "dtm", desc: "<code>dtm</code> (Gelände) oder <code>dsm</code> (Oberfläche)." },
]} />

Die Antwort ist ein GeoJSON-`Feature` mit `LineString` (Koordinaten mit Höhe als dritter Wert). In `properties` stehen `distances` (Meter ab Linienanfang) und `elevations` je Messpunkt sowie `distance_m`, `ascent_m`, `descent_m`, `min_m`, `max_m`, `model`, `stichtag` und `resolution_m`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.70,47.08&samples=50"`} />

Live: [Höhenprofil](/beispiele/hoehenprofil/), [Höhe per Klick](/beispiele/hoehe-abfragen/).

## Fehler

Fehler kommen als `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)).

| Status | Ursache |
|---|---|
| `400` | weder oder beide von <code>points</code>/<code>path</code>, Koordinate nicht <code>lon,lat</code>, zu viele Punkte, <code>samples</code> oder <code>model</code> ungültig |
| `429` | Limit überschritten, <code>Retry-After</code> beachten |
| `503` | vorübergehend nicht verfügbar |

<Code lang="json" code={`{ "type": "about:blank", "title": "Bad Request", "status": 400, "detail": "model: dtm oder dsm" }`} />

## Limits

- Bis 512 Punkte je Anfrage (<code>points</code> oder Stützpunkte von <code>path</code>), bis 1024 Profilpunkte.
- Anfragelimit je IP-Adresse; große Anfragen zählen mehr. Details: [Fehler und Limits](/api/fehler/).
- Maschinenlesbar: [openapi.json](https://api.mapx.at/v1/openapi.json).
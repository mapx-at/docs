# Wetter

Aktuelle Messwerte, stündliche Punktprognose und Wetterkacheln aus offenen Daten der GeoSphere Austria.

Die Wetter-API liefert aktuelle Messwerte der nächsten Wetterstation, eine stündliche Prognose für einen Punkt und fertig eingefärbte Rasterkacheln für Temperatur, Niederschlag, Wind, Feuchte, Bewölkung und Luftdruck. Kein Schlüssel, CORS offen.

> Alle Wetterdaten stammen von der **GeoSphere Austria** (früher ZAMG) und stehen unter [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Wer die Daten oder daraus gerenderte Kacheln zeigt, muss die Quelle nennen, zum Beispiel „Wetter: GeoSphere Austria, CC BY 4.0“. Jede Antwort enthält dazu das Feld `attribution` bzw. den Header `X-Weather-Attribution`, das TileJSON das Feld `attribution`.

Die Daten werden alle 10 Minuten (Messwerte) bzw. mit jedem neuen Modelllauf (Prognose, etwa alle 3 Stunden, Auflösung 1 km) zentral geholt und zwischengespeichert. Anfragen an die mapx-API belasten die Quelle nicht.

## Aktuelle Messwerte

**GET https://api.mapx.at/v1/weather/current**

<ParamTable params={[
  { name: "lon", type: "number", required: true, desc: "Längengrad (WGS84), auch <code>lng</code>." },
  { name: "lat", type: "number", required: true, desc: "Breitengrad." },
  { name: "limit", type: "integer", default: "3", desc: "Anzahl der nächsten Stationen, 1 bis 10." },
  { name: "radius", type: "number", default: "50", desc: "Suchradius in Kilometern." },
]} />

Antwort: GeoJSON `FeatureCollection`, nächste Station zuerst (TAWES-Messnetz der GeoSphere, rund 280 Stationen). Fehlt ein Messwert an der Station, ist er `null`.

<ParamTable kind="response" params={[
  { name: "station_id", type: "string", desc: "Stationsnummer (Synop)." },
  { name: "name, state, altitude_m", type: "string, string, number", desc: "Name, Bundesland, Seehöhe der Station." },
  { name: "distance_m", type: "number", desc: "Entfernung zum Punkt in Metern." },
  { name: "time", type: "string", desc: "Messzeitpunkt, UTC." },
  { name: "temperature_c", type: "number", desc: "Lufttemperatur in °C." },
  { name: "humidity_pct", type: "number", desc: "Relative Feuchte in %." },
  { name: "wind_speed_ms, wind_speed_kmh, wind_gust_ms, wind_direction_deg", type: "number", desc: "Mittelwind (10 Minuten), Spitze, Richtung, aus der der Wind weht." },
  { name: "precipitation_mm_10min", type: "number", desc: "Niederschlag der letzten 10 Minuten in mm." },
  { name: "pressure_hpa, pressure_msl_hpa", type: "number", desc: "Luftdruck an der Station und auf Meeresniveau reduziert." },
]} />

<Code lang="bash" code={`# Melk
curl "https://api.mapx.at/v1/weather/current?lon=15.334&lat=48.227&limit=1"`} />

## Prognose

**GET https://api.mapx.at/v1/weather/forecast**

<ParamTable params={[
  { name: "lon", type: "number", required: true, desc: "Längengrad (WGS84)." },
  { name: "lat", type: "number", required: true, desc: "Breitengrad." },
  { name: "hours", type: "integer", default: "24", desc: "Anzahl Stunden ab der aktuellen Stunde, 1 bis 60 (je nach Modelllauf stehen weniger zur Verfügung)." },
]} />

Die Werte kommen aus dem Modell AROME (`nwp-v2-1h-1km`) und sind bilinear zum Punkt interpoliert. Gebiet: Österreich mit Rand, außerhalb `404`.

<Code lang="json" code={`{
  "location": { "lon": 15.334, "lat": 48.227, "grid_lon": 15.3347, "grid_lat": 48.225 },
  "model": "GeoSphere AROME nwp-v2-1h-1km",
  "reference_time": "2026-11-07T12:00Z",
  "interval": "1h",
  "hours": [
    { "time": "2026-11-07T20:00Z", "temperature_c": 4.7, "precipitation_mm": 0.0, "wind_speed_ms": 4.1,
      "wind_direction_deg": 52.0, "humidity_pct": 69.9, "cloud_cover_pct": 20.0, "pressure_msl_hpa": 1027.2 }
  ],
  "attribution": "Datenquelle: GeoSphere Austria, CC BY 4.0 (https://data.hub.geosphere.at)"
}`} />

`precipitation_mm` ist die Summe der jeweils letzten Stunde. Alle Zeiten sind UTC.

## Kacheln

**GET https://api.mapx.at/v1/weather/tiles/{parameter}/{time}/{z}/{x}/{y}.png**

PNG-Kacheln 256 × 256, halbtransparent, außerhalb des Prognosegebiets leer. Zoom 0 bis 14 (Daten 1 km, ab etwa Zoom 12 nur noch glatt interpoliert).

<ParamTable params={[
  { name: "parameter", type: "string", required: true, desc: "Größe.", values: [
    { v: "temperature", d: "Lufttemperatur 2 m, °C" },
    { v: "precipitation", d: "Niederschlag je Stunde, mm/h" },
    { v: "wind", d: "Windgeschwindigkeit 10 m, m/s" },
    { v: "humidity", d: "Relative Feuchte, %" },
    { v: "cloud", d: "Bewölkung, %" },
    { v: "pressure", d: "Luftdruck auf Meeresniveau, hPa" },
  ] },
  { name: "time", type: "string", required: true, desc: "<code>now</code> (nächste Stunde) oder eine UTC-Stunde wie <code>2026-11-07T20:00Z</code>. Außerhalb der Prognose <code>404</code>." },
]} />

### TileJSON mit Zeitdimension

**GET https://api.mapx.at/v1/weather/tiles/{parameter}.json**

Das TileJSON 3.0.0 hat `tiles` für `now`, `bounds`, `attribution` und die Erweiterungen:

- `mapx:time.times`: die verfügbaren Stunden (UTC), `mapx:time.reference_time`: der Modelllauf.
- `mapx:time.template`: Kachel-URL mit Platzhalter `{time}`.
- `mapx:legend`: Einheit und Farbstützstellen der Skala.

Kacheln mit fester Stunde sind eine Stunde zwischenspeicherbar, mit `now` fünf Minuten.

<Tabs syncKey="lang">
  <TabItem label="JavaScript">
<Code lang="js" code={`const API = "https://api.mapx.at/v1";
const tj = await (await fetch(API + "/weather/tiles/temperature.json")).json();
const t = tj["mapx:time"].times[3];            // vierte Stunde
map.addSource("wetter", { type: "raster", tileSize: 256, maxzoom: 12,
  tiles: [tj["mapx:time"].template.replace("{time}", t)],
  attribution: "Wetter: GeoSphere Austria, CC BY 4.0" });
map.addLayer({ id: "wetter", type: "raster", source: "wetter", paint: { "raster-opacity": 0.85 } });`} />
  </TabItem>
  <TabItem label="curl">
<Code lang="bash" code={`curl "https://api.mapx.at/v1/weather/tiles/temperature.json"
curl -o kachel.png "https://api.mapx.at/v1/weather/tiles/precipitation/now/8/137/89.png"`} />
  </TabItem>
</Tabs>

Live: [Wetter-Overlay](/beispiele/wetter-overlay/).

## Fehler

Fehler kommen als `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)).

| Status | Ursache |
|---|---|
| `400` | `lon`/`lat` fehlen oder sind keine Zahlen, `hours`/`limit` außerhalb des Bereichs, `time` nicht lesbar |
| `404` | Punkt außerhalb des Prognosegebiets, Parameter unbekannt, Zeit außerhalb der Prognose |
| `429` | Limit überschritten, `Retry-After` beachten |
| `503` | Daten noch nicht geladen (kurz nach einem Neustart) |

## Lizenz und Herkunft

GeoSphere Austria Data Hub, Datensätze `tawes-v1-10min` (Stationsmessungen) und `nwp-v2-1h-1km` (AROME). Lizenz [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de), die Nutzung ist auch kommerziell erlaubt. Die Zahlen werden gerundet bzw. interpoliert und farbig dargestellt, das ist eine Bearbeitung der Originaldaten. Verbindliche Warnungen und Vorhersagen bietet die GeoSphere Austria unter [geosphere.at](https://www.geosphere.at/).
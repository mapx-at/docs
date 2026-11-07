# Öffentlicher Verkehr

Verbindungsauskunft mit Fußweg oder Rad zur Haltestelle, Abfahrtsmonitor und Haltestellen aus den offenen GTFS-Fahrplänen von ÖBB und Verkehrsverbünden.

Die Auskunft verbindet den Fahrplan (GTFS, Routing-Verfahren RAPTOR) mit dem Fuß- und Radnetz [GIP.at](https://www.gip.at). Zubringer, Umstiegswege und Abgang werden auf dem echten Wegenetz berechnet. Fahrplandaten: ÖBB-Personenverkehr AG, [Soll-Fahrplan](https://data.oebb.at/de/datensaetze~soll-fahrplan-gtfs~) unter CC BY 4.0. Dazu kommen die Fahrpläne der Verkehrsverbünde (VOR, Verbundlinie, Salzburg, Kärnten, OÖVV, VVT, Vmobil, Linz AG) von [data.mobilitaetsverbuende.at](https://data.mobilitaetsverbuende.at) unter der Datenlizenz Mobilitätsverbünde Österreich: Stadt-, Regional- und Landbusse, Straßen- und U-Bahnen. Das Fahrplanjahr 2026 gilt vom 14.12.2025 bis 12.12.2026, andere Tage liefern `422`.

## Verbindungen

**GET https://api.mapx.at/v1/directions/transit/{coordinates}**

Rad zur Haltestelle: `GET /v1/directions/bike_transit/{coordinates}`.

<ParamTable params={[
  { name: "coordinates", type: "string", required: true, desc: "Genau Start und Ziel als <code>lon,lat;lon,lat</code>." },
  { name: "depart_at", type: "string", desc: "Frühestens ab, ISO 8601 in Wiener Ortszeit, etwa <code>2026-11-09T08:00</code>. Auch mit Offset (<code>Z</code>, <code>+01:00</code>). Standard: jetzt." },
  { name: "arrive_by", type: "string", desc: "Spätestens an, statt <code>depart_at</code>. Die Suche läuft rückwärts, die erste Verbindung fährt so spät wie möglich." },
  { name: "routes", type: "integer", default: "3", desc: "Anzahl der Verbindungen, 1 bis 5." },
]} />

Profile: `transit` (zu Fuß bis 20 Minuten zur Haltestelle) und `bike_transit` (Rad bis 15 Minuten, das Rad wird nicht im Zug mitgenommen). Umstiege zwischen Haltestellen sind immer Fußwege bis 8 Minuten, an derselben Station gelten mindestens 2 Minuten.

Antwort: `code`, `routes[]`, `waypoints[]`, `query` und `attribution`. Jede Route hat `duration` (Sekunden), `distance` (Meter), `departure`, `arrival` (ISO 8601), `transfers`, `summary`, `geometry` (GeoJSON) und `legs[]`. Jeder Abschnitt hat `mode`:

- `walking` oder `cycling`: `duration`, `distance`, `from`, `to`, `geometry`. Der erste Abschnitt endet mit der Abfahrt, der letzte beginnt mit der Ankunft.
- `transit`: `line` (`name` wie „RJX 61“, `long_name`, `type` wie `rail` oder `bus`, `agency`, `headsign`), `from`, `to`, `stops[]` (mit Ankunft und Abfahrt je Halt) und `geometry` als Linie durch die Halte. Die Linie folgt dem Linienverlauf aus dem Liniennetz der Mobilitätsverbünde, wo es einen kennt, sonst den Haltestellen.

Liegt das Ziel nahe, enthält `routes[]` auch den reinen Fußweg (bzw. Radweg).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/transit/15.3338,48.2279;16.3770,48.1850?depart_at=2026-11-09T08:00"`} />

## Abfahrtsmonitor

**GET https://api.mapx.at/v1/departures**

<ParamTable params={[
  { name: "stop", type: "string", required: true, desc: "Stations-ID aus <code>/v1/stops</code> oder der Suche (<code>stop_id</code>), etwa <code>at:43:7226</code> (IFOPT, auch eine Steig-ID wie <code>at:43:7226:0:2</code>), oder <code>lon,lat</code>: dann alle Stationen im Umkreis <code>radius</code>." },
  { name: "time", type: "string", desc: "Ab wann, ISO 8601 in Wiener Ortszeit oder Unix-Sekunden. Standard: jetzt." },
  { name: "limit", type: "integer", default: "20", desc: "Anzahl Abfahrten, 1 bis 100." },
  { name: "radius", type: "integer", default: "300", desc: "Nur bei <code>lon,lat</code>: Umkreis in Metern, 1 bis 2000." },
  { name: "horizon", type: "integer", default: "14400", desc: "Zeitfenster in Sekunden ab <code>time</code>, 60 bis 86400." },
]} />

Nächste Abfahrten nach dem **Soll-Fahrplan**: Verkehrstage nach Kalender und Fahrplanperiode (inklusive Ausnahmetagen und Fahrten, die nach Mitternacht vom Vortag weiterlaufen). Echtzeitdaten gibt es nicht, `realtime` ist immer `false`. Am Endhalt einer Fahrt gibt es keine Abfahrt.

Antwort: `time`, `realtime`, `stops[]` (die gewählten Stationen) und `departures[]`, je Abfahrt `time`, `scheduled`, `realtime`, `line` (`name`, `short_name`, `long_name`, `trip_short_name` wie „R 2004“, `mode` wie `rail`, `bus`, `tram`, `subway`, `route_type`), `headsign`, `destination` (`id`, `name`), `platform` (Steig, `null` wenn unbekannt), `operator`, `stop` und `trip_id`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/departures?stop=at:43:7226&time=2026-11-09T08:00&limit=5"`} />

```json
{"time":"2026-11-09T08:00:00+01:00","realtime":false,
 "departures":[{"time":"2026-11-09T08:00:00+01:00","realtime":false,
   "line":{"name":"R52","trip_short_name":"R 2004","mode":"rail","route_type":2},
   "headsign":"Pöchlarn Bahnhof","platform":"2","operator":"OEBB Personenverkehr AG Kundenservice",
   "stop":{"id":"at:43:7226","name":"Melk Bahnhof"}}]}
```

## Haltestellen

**GET https://api.mapx.at/v1/stops**

<ParamTable params={[
  { name: "bbox", type: "string", desc: "<code>west,süd,ost,nord</code>, höchstens 1 Quadratgrad. Größte Knoten zuerst." },
  { name: "near", type: "string", desc: "<code>lon,lat</code>, statt <code>bbox</code>: nächste zuerst, mit <code>distance</code> in Metern." },
  { name: "radius", type: "integer", default: "500", desc: "Nur mit <code>near</code>: Meter, 1 bis 5000." },
  { name: "limit", type: "integer", default: "50", desc: "1 bis 500." },
]} />

Stationen aus allen Feeds; Steige und die Einträge verschiedener Verbünde mit derselben IFOPT-Kennung (`at:<land>:<nr>`) sind zu einer Station zusammengeführt. Je Station `id`, `name`, `location` (`[lon, lat]`), `modes` und `lines`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/stops?near=16.3752,48.1851&radius=300"`} />

## Fehler

| Status | Bedeutung |
|---|---|
| 400 | Koordinaten oder Zeit ungültig, <code>depart_at</code> und <code>arrive_by</code> zugleich |
| 404 | keine Haltestelle in Reichweite (auch unbekannte <code>stop</code>), oder keine Verbindung innerhalb von 48 Stunden |
| 422 | Datum außerhalb des Fahrplanzeitraums |

Verbindungen werden nicht gecacht, Abfahrten 30 Sekunden, Haltestellen eine Stunde. Quelle angeben: „Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)“.
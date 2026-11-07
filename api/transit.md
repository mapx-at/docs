# Öffentlicher Verkehr

Verbindungsauskunft mit Fußweg oder Rad zur Haltestelle und Fahrplan aus den offenen GTFS-Daten der ÖBB.

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
- `transit`: `line` (`name` wie „RJX 61“, `long_name`, `type` wie `rail` oder `bus`, `agency`, `headsign`), `from`, `to`, `stops[]` (mit Ankunft und Abfahrt je Halt) und `geometry` als Linie durch die Halte. Die Linie folgt den Haltestellen, nicht dem Gleis.

Liegt das Ziel nahe, enthält `routes[]` auch den reinen Fußweg (bzw. Radweg).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/directions/transit/15.3338,48.2279;16.3770,48.1850?depart_at=2026-11-09T08:00"`} />

## Fehler

| Status | Bedeutung |
|---|---|
| 400 | Koordinaten oder Zeit ungültig, <code>depart_at</code> und <code>arrive_by</code> zugleich |
| 404 | keine Haltestelle in Reichweite, oder keine Verbindung innerhalb von 48 Stunden |
| 422 | Datum außerhalb des Fahrplanzeitraums |

Die Antworten werden nicht gecacht. Quelle angeben: „Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)“.
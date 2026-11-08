# Transit-Abfahrten

Verbindungen, Haltestellen und Abfahrtstafeln im öffentlichen Verkehr aus dem Soll-Fahrplan.

Die Fahrplandaten decken Österreich ab (ÖBB und die regionalen Verkehrsverbünde). Es ist der Soll-Fahrplan: `realtime` ist immer `false`. Anfragen für Verbindungen liegen in der Zone `route`, für Haltestellen und Abfahrten in der Zone `search`.

| Aufgabe | Endpunkt |
|---|---|
| Haltestellen in der Nähe eines Punkts finden | [`/v1/stops`](/de/api/transit/) |
| Nächste Abfahrten an einer Haltestelle | [`/v1/departures`](/de/api/transit/) |
| Verbindung von A nach B | [`/v1/directions/transit/{coordinates}`](/de/api/transit/) |

## Haltestellen finden

```bash
curl -s "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq '.stops[] | {id, name, modes, lines}'
```

Haltestellen derselben Station von verschiedenen Betreibern werden zusammengeführt und teilen sich eine `id` (zum Beispiel `at:49:1320`). Mit `bbox` (höchstens ein Quadratgrad) lässt sich ein Kartenausschnitt auflisten.

## Abfahrtstafel

Eine Haltestellen-`id` übergeben oder eine Koordinate, um die Abfahrten aller Haltestellen in der Umgebung zu erhalten.

```js
const url = "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=8";
const { departures } = await (await fetch(url)).json();
for (const d of departures) console.log(d.time.slice(11, 16), d.line.name, d.headsign, d.platform ?? "");
```

`time` ist ISO 8601 in Wiener Ortszeit. Mit `time=2026-11-09T08:00` lässt sich ein späterer Start setzen, mit `horizon` (Sekunden, bis zu 24 Stunden) das Zeitfenster. Außerhalb des Fahrplanzeitraums antwortet die API mit `422`.

## Verbindungssuche

```bash
curl -s "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00&routes=3" | jq '.routes[0].duration'
```

Genau einen Start und ein Ziel übergeben. `depart_at` (früheste Abfahrt) oder `arrive_by` (späteste Ankunft) verwenden, Standard ist jetzt. Die Antwort enthält Fußwege auf dem Straßennetz und Fahrten mit Linie, Haltestellen und Zeiten.

## Abfahrten in einer App zeigen

1. Eine Haltestelle mit [`/v1/stops`](/de/api/transit/) suchen oder die Nutzer auf die Karte klicken lassen.
2. `/v1/departures` aufrufen und jede Minute aktualisieren.
3. Linie, Ziel und `time` anzeigen; fehlende Echtzeitdaten mit "planmäßig" kennzeichnen.

## Nächste Schritte

- [Transit API](/de/api/transit/)
- [Navigation](/de/guides/navigation/)
- [Attribution](/de/reference/attribution/): Quellenangabe der Fahrplandaten
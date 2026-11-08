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
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq '.stops[] | {id, name, modes, lines}'
```

Haltestellen derselben Station von verschiedenen Betreibern werden zusammengeführt und teilen sich eine `id` (zum Beispiel `at:49:1320`). Mit `bbox` (höchstens ein Quadratgrad) lässt sich ein Kartenausschnitt auflisten.

## Abfahrtstafel

Eine Haltestellen-`id` übergeben oder eine Koordinate, um die Abfahrten aller Haltestellen in der Umgebung zu erhalten.

```js
const url = "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=8";
const { departures } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
for (const d of departures) console.log(d.time.slice(11, 16), d.line.name, d.headsign, d.platform ?? "");
```

`time` ist ISO 8601 in Wiener Ortszeit. Mit `time=2026-11-09T08:00` lässt sich ein späterer Start setzen, mit `horizon` (Sekunden, bis zu 24 Stunden) das Zeitfenster. Außerhalb des Fahrplanzeitraums antwortet die API mit `422`.

## Verbindungssuche

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00&routes=3" | jq '.routes[0].duration'
```

Genau einen Start und ein Ziel übergeben. `depart_at` (früheste Abfahrt) oder `arrive_by` (späteste Ankunft) verwenden, Standard ist jetzt. Die Antwort enthält Fußwege auf dem Straßennetz und Fahrten mit Linie, Haltestellen und Zeiten.

## Erreichbarkeit mit ÖV

Zu Fuß plus öffentlicher Verkehr als Isochronen: Profil `transit` der Isochronen-API. Die Suche geht zu den Haltestellen rund um den Start, fährt ab `depart_at` nach Soll-Fahrplan und geht von jeder erreichten Haltestelle wieder zu Fuß weiter.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/isochrone/transit/16.3725,48.2085?contours_minutes=10,20,30&depart_at=2026-11-10T08:00" | jq '.features[] | {contour: .properties.contour, km2: .properties.area_km2}'
```

Ohne `depart_at` beginnt die Isochrone jetzt (Wiener Ortszeit). Es gilt nur `contours_minutes`. Die Fläche hängt stark von der Uhrzeit ab: derselbe Start hat nachts eine deutlich kleinere Fläche. Außerhalb des Fahrplanzeitraums antwortet die API mit `422`.

### Zugang mit Rad oder Auto (Bike & Ride, Park & Ride)

Mit `access=bike` oder `access=car` wird der erste Abschnitt nicht gegangen, sondern mit dem Rad oder Auto zu einer Haltestelle zurückgelegt. Das Rad wird an **jeder** Haltestelle abgesperrt (+2 min), das Auto parkt an Haltestellen mit Bahn-, U-Bahn- oder Tramverkehr (+3 min, höchstens 25 min Fahrt); der Algorithmus nimmt die schnellste Kombination, der Bahnhof im eigenen Ort gewinnt also gegen die Weiterfahrt in die nächste Stadt, wenn der Zug früher fährt. Offizielle P+R-Anlagen der GIP kommen fürs Auto dazu (+2 min, dann Fußweg zur Haltestelle). Der Fußweg vom Start gilt weiter, es zählt das Minimum. Der Abgang am Ziel ist immer zu Fuß.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/isochrone/transit/15.16,48.207?contours_minutes=45&depart_at=2026-11-09T07:00&access=car" | jq '.metadata | {access, access_stops_improved, sites: .access_sites[:3]}'
```

| Parameter | Werte | Bedeutung |
|---|---|---|
| `access` | `foot` (Standard), `bike`, `car` | Verkehrsmittel zur ersten Haltestelle |
| `park` | `any` (Standard), `official` | `official`: Rad nur an Haltestellen im Umkreis von 200 m einer B+R-Anlage, Auto nur an offiziellen P+R-Anlagen |
| `sites` | 1 bis 50 (Standard 10) | Einträge in `metadata.access_sites` |

`metadata.access_sites` nennt die Zugänge, die den Fußweg schlagen, sortiert nach Fahrzeit: `source` (`site` = offizielle P+R-Anlage, `stop` = Parken an der Haltestelle), `kind`, `name`, `lon`, `lat`, `arrive_min` (Fahrzeit), `ready_min` (bereit zum Einsteigen inkl. Parken und Fußweg) und `stops_won`. Rad im Zug (`egress=bike`) gibt es nicht: die Fahrplandaten enthalten keine Fahrradmitnahme-Information.

## Abfahrten in einer App zeigen

1. Eine Haltestelle mit [`/v1/stops`](/de/api/transit/) suchen oder die Nutzer auf die Karte klicken lassen.
2. `/v1/departures` aufrufen und jede Minute aktualisieren.
3. Linie, Ziel und `time` anzeigen; fehlende Echtzeitdaten mit „planmäßig“ kennzeichnen.

## Nächste Schritte

- [Transit API](/de/api/transit/)
- [Navigation](/de/guides/navigation/)
- [Attribution](/de/reference/attribution/): Quellenangabe der Fahrplandaten
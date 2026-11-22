# ÖV-Guide: Abfahrten und Verbindungen

Source: https://docs.mapx.at/de/guides/transit/

> ÖV-Verbindungen, Haltestellen und Abfahrtstafeln in Österreich aus dem Soll-Fahrplan mit der mapx-ÖV-API abfragen und in der Karte anzeigen.

Die Fahrplandaten decken Österreich ab (ÖBB und die regionalen Verkehrsverbünde). Es ist der Soll-Fahrplan: `realtime` ist immer `false`. Anfragen für Verbindungen liegen in der Zone `route`, für Haltestellen und Abfahrten in der Zone `search`. Jede Anfrage braucht einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Beispiele senden ihn im Header `Authorization`, den auch Browser senden dürfen.

| Aufgabe | Endpunkt |
|---|---|
| Haltestellen in der Nähe eines Punkts finden | [`/v1/stops`](https://docs.mapx.at/de/api/transit.md) |
| Nächste Abfahrten an einer Haltestelle | [`/v1/departures`](https://docs.mapx.at/de/api/transit.md) |
| Verbindung von A nach B | [`/v1/directions/transit/{coordinates}`](https://docs.mapx.at/de/api/transit.md) |

## Haltestellen finden

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq '.stops[] | {id, name, modes, lines}'
```

Haltestellen derselben Station von verschiedenen Betreibern werden zusammengeführt und teilen sich eine `id` (zum Beispiel `at:49:1320`). Mit `bbox` (höchstens ein Quadratgrad) lässt sich ein Kartenausschnitt auflisten.

## Abfahrtstafel

Eine Haltestellen-`id` übergeben oder eine Koordinate, um die Abfahrten aller Haltestellen in der Umgebung zu erhalten. Eine Steig-Id wie `at:43:7226:0:2` liefert nur die Abfahrten von diesem Steig (`stops[0]` hat dann `platform_id` und `platform`), die Stations-Id alle Steige.

```js
const url = "https://api.mapx.at/v1/departures?stop=at:49:1320&limit=8";
const { departures } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
for (const d of departures) console.log(d.time.slice(11, 16), d.line.name, d.headsign, d.platform ?? "");
```

`time` ist ISO 8601 in Wiener Ortszeit. Ohne `time` gilt jetzt. Mit `time` (Format `JJJJ-MM-TTThh:mm`, Wiener Ortszeit) lässt sich ein späterer Start setzen, mit `horizon` (Sekunden, bis zu 24 Stunden) das Zeitfenster. Der Fahrplan gilt nur für einen begrenzten Zeitraum: `timetable.valid_from` und `timetable.valid_to` in jeder Antwort nennen ihn (rund um den Fahrplanwechsel im Dezember sind beide Fahrplanjahre zusammen geladen). Außerhalb davon antwortet die API mit `422`, und die Meldung (`detail`) nennt den gültigen Zeitraum. Ungültige Daten oder Uhrzeiten wie `2026-02-31` oder `T25:00` ergeben `400`. An den Tagen der Zeitumstellung stimmen die Zeiten; eine Ortszeit, die es nicht gibt (im Frühjahr zwischen 02:00 und 03:00), rückt in die Sommerzeit vor.

`line.name` ist der Linienname für Fahrgäste, in Abfahrten und Verbindungen gleich: `U1`, `S50`, `R52`, Busnummern, Fernzüge mit Zugnummer wie `RJ 754`. `trip_short_name` enthält die Zugnummer.

## Verbindungssuche

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855" | jq '.routes[0].duration'
```

Genau einen Start und ein Ziel übergeben. `depart_at` (früheste Abfahrt) oder `arrive_by` (späteste Ankunft) verwenden, jeweils im Format `JJJJ-MM-TTThh:mm` in Wiener Ortszeit; Standard ist jetzt. Die Antwort enthält mehrere Verbindungen in `routes[]` (Anzahl wählbar mit `routes`, siehe [ÖV-API](https://docs.mapx.at/de/api/transit.md)), dazu Fußwege auf dem Straßennetz und Fahrten mit Linie, Haltestellen, Zeiten und Steig (`from.platform`, `to.platform`, `stops[].platform`, `null`, wenn unbekannt). Umsteigen an derselben Station dauert mindestens 2 Minuten, nach einem Fußweg bleibt mindestens 1 Minute bis zur Abfahrt. Mit `bike_transit` ist der erste Abschnitt mit dem Rad; das Rad bleibt an der Haltestelle, Umstiege und der letzte Abschnitt sind zu Fuß. Keine Haltestelle in Reichweite oder keine Verbindung: `422`.

## Nach Verkehrsmittel filtern

`/v1/stops`, `/v1/departures` und beide Verbindungsprofile nehmen `modes`, eine durch Komma getrennte Liste von Verkehrsmitteln. Ohne `modes` gelten alle. Dieselben Werte stehen in den Antworten: `stops[].modes`, `departures[].line.mode` sowie `mode` und `line.mode` jeder Fahrt einer Verbindung (Fußwege sind `walking` bzw. `cycling`).

| Wert | Umfasst (GTFS `route_type`) |
|---|---|
| `rail` | Fern-, Regional- und S-Bahn-Züge (2, 100–199, 300–399) |
| `subway` | U-Bahn, Metro (1, 400–404) |
| `tram` | Straßenbahn (0, 900–999) |
| `bus` | Bus inkl. Obus, Fernbus und Rufbus (3, 11, 200–299, 700–899, 1500–1599) |
| `ferry` | Schiff, Fähre (4, 1000–1099, 1200–1299) |
| `cable` | Seil- und Kabinenbahn, Sessellift, Standseilbahn (5–7, 1300–1499) |
| `other` | Alles andere (in den aktuellen Fahrplandaten nicht vorhanden) |

```bash
# nur die U-Bahn am Stephansplatz
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/departures?stop=at:49:1320&modes=subway&limit=8" | jq '.departures[] | {time, line: .line.name, mode: .line.mode}'
# Wien Hauptbahnhof nach St. Pölten nur mit dem Bus (ohne Bahn)
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3772,48.1852;15.6256,48.2049?modes=bus&routes=1" | jq '.routes[0].legs[] | {mode, line: .line.name}'
```

`stops` liefert dann nur Haltestellen, an denen eines dieser Verkehrsmittel hält, `modes` und `lines` nennen nur diese; mit einer Koordinate sucht `departures` nur solche Haltestellen. Verbindungen nutzen nur Fahrten mit diesen Verkehrsmitteln, Fußwege und der Radabschnitt bleiben. Eine Verbindung hat höchstens 8 Fahrten, über weite Strecken findet sich daher mit `bus` allein womöglich keine Verbindung (`422`). Ein unbekannter Wert ergibt `400` mit `param: "modes"` und den erlaubten Werten in `allowed`.

## Erreichbarkeit mit ÖV

Zu Fuß plus öffentlicher Verkehr als Isochronen: Profil `transit` der Isochronen-API. Die Suche geht zu den Haltestellen rund um den Start, fährt ab `depart_at` nach Soll-Fahrplan und geht von jeder erreichten Haltestelle wieder zu Fuß weiter.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/isochrone/transit/16.3725,48.2085?contours_minutes=10,20,30" | jq '.features[] | {contour: .properties.contour, km2: .properties.area_km2}'
```

Ohne `depart_at` beginnt die Isochrone jetzt (Wiener Ortszeit); mit `depart_at` wählst du eine andere Zeit. Es gilt nur `contours_minutes`. Die Fläche hängt stark von der Uhrzeit ab: derselbe Start hat nachts eine deutlich kleinere Fläche. Außerhalb des Fahrplanzeitraums antwortet die API mit `422`.

### Zugang mit Rad oder Auto (Bike & Ride, Park & Ride)

Mit `access=bike` oder `access=car` wird der erste Abschnitt nicht gegangen, sondern mit dem Rad oder Auto zu einer Haltestelle zurückgelegt. Das Rad wird an **jeder** Haltestelle abgesperrt (+2 min), das Auto parkt an Haltestellen mit Bahn-, U-Bahn- oder Tramverkehr (+3 min, höchstens 25 min Fahrt); der Algorithmus nimmt die schnellste Kombination, der Bahnhof im eigenen Ort gewinnt also gegen die Weiterfahrt in die nächste Stadt, wenn der Zug früher fährt. Ausgewiesene P+R-Anlagen der GIP kommen fürs Auto dazu (+2 min, dann Fußweg zur Haltestelle). Der Fußweg vom Start gilt weiter, es zählt das Minimum. Der Abgang am Ziel ist immer zu Fuß.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/isochrone/transit/15.16,48.207?contours_minutes=45&access=car" | jq '.metadata | {access, access_stops_improved, sites: .access_sites[:3]}'
```

| Parameter | Werte | Bedeutung |
|---|---|---|
| `access` | `foot` (Standard), `bike`, `car` | Verkehrsmittel zur ersten Haltestelle |
| `park` | `any` (Standard), `official` | `official`: Rad nur an Haltestellen im Umkreis von 200 m einer B+R-Anlage, Auto nur an ausgewiesenen P+R-Anlagen |
| `sites` | 1 bis 50 (Standard 10) | Einträge in `metadata.access_sites` |

`metadata.access_sites` nennt die Zugänge, die den Fußweg schlagen, sortiert nach Fahrzeit: `source` (`site` = ausgewiesene P+R-Anlage, `stop` = Parken an der Haltestelle), `kind`, `name`, `lon`, `lat`, `arrive_min` (Fahrzeit), `ready_min` (bereit zum Einsteigen inkl. Parken und Fußweg) und `stops_won`. Rad im Zug (`egress=bike`) gibt es nicht: die Fahrplandaten enthalten keine Fahrradmitnahme-Information.

## Abfahrten in einer App zeigen

1. Eine Haltestelle mit [`/v1/stops`](https://docs.mapx.at/de/api/transit.md) suchen oder die Nutzer auf die Karte klicken lassen.
2. `/v1/departures` aufrufen und jede Minute aktualisieren.
3. Linie, Ziel und `time` anzeigen; die Zeiten als „planmäßig“ kennzeichnen (Soll-Fahrplan).

## Nächste Schritte

- [Transit API](https://docs.mapx.at/de/api/transit.md)
- [Navigation](https://docs.mapx.at/de/guides/navigation.md)
- [Quellenangabe](https://docs.mapx.at/de/reference/attribution.md): Quellenangabe der Fahrplandaten

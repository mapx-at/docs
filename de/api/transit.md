# ÖV-API: Verbindungen und Abfahrten

Source: https://docs.mapx.at/de/api/transit/

> ÖV-API für den öffentlichen Verkehr in Österreich: Verbindungen, Haltestellen und Abfahrtstafeln aus den Fahrplandaten, mit Parametern und Beispielen.

Verbindungen mit öffentlichem Verkehr (zu Fuß oder mit dem Rad zur Haltestelle), Abfahrtstafeln und Haltestellen auf Basis des Fahrplans.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `route` (5/s), `search` (10/s) · Guide: [ÖV-Abfahrten](https://docs.mapx.at/de/guides/transit.md)

## Öffentlicher Verkehr: zu Fuß und mit Öffis (GTFS-Fahrplan, RAPTOR) {#public-transit-walking-plus-transit-gtfs-timetable-raptor}

`GET /v1/directions/transit/{coordinates}`

Genau Start und Ziel. Verbindungen mit Fußwegen auf dem GIP.at-Netz und Fahrten (Linie, Haltestellen, Zeiten, `platform` bei `from`, `to` und `stops[]`, wenn bekannt, sonst null). Zeit über `depart_at` (Standard jetzt) oder `arrive_by`, ISO 8601 in Wiener Ortszeit; ungültige Daten oder Uhrzeiten wie `T25:00` oder `2026-02-31` ergeben 400.

`line.name` ist der Linienname für Fahrgäste wie in `/v1/departures` (`U3`, `S50`, `R52`, Fernzüge mit Zugnummer wie `RJ 754`), `trip_short_name` die Zugnummer. Umsteigen an derselben Station dauert mindestens 2 Minuten, nach einem Fußweg bleibt mindestens 1 Minute bis zur Abfahrt. Höchstens `routes` Verbindungen, sortiert nach Ankunft (`depart_at`) bzw. nach Abfahrt, späteste zuerst (`arrive_by`); Verbindungen, die nicht besser als eine andere sind (nicht früher, nicht schneller, nicht weniger Fahrten), und Flügelzüge mit gleichen Zeiten erscheinen nur einmal. Ein direkter Fußweg steht vorne, wenn er nicht langsamer als die beste Fahrt ist. `timetable` nennt den Zeitraum des geladenen Fahrplans (`valid_from`, `valid_to`); außerhalb davon und wenn keine Haltestelle erreichbar ist oder keine Verbindung gefunden wird 422. Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG, data.mobilitaetsverbuende.at.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | path | string | ja | – | lon,lat;lon,lat (genau Start und Ziel) |
| `depart_at` | query | string |  | – | Früheste Abfahrt, z. B. 2026-11-09T08:00 |
| `arrive_by` | query | string |  | – | Späteste Ankunft, statt depart_at |
| `routes` | query | string |  | `3` | 1 bis 5, Standard 3 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "attribution": "mapx.at; Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)",
  "code": "Ok",
  "query": {
    "depart_at": "2026-11-23T08:00:00+01:00",
    "profile": "transit"
  },
  "routes": [
    {
      "arrival": "2026-11-23T08:15:51+01:00",
      "departure": "2026-11-23T08:01:26+01:00",
      "distance": 4801,
      "duration": 865,
      "geometry": {
        "coordinates": [
          [
            16.3725,
            48.2085
          ]
        ],
        "type": "LineString"
      },
      "legs": [
        {
          "arrival": "2026-11-23T08:03:00+01:00",
          "departure": "2026-11-23T08:01:26+01:00",
          "distance": 117,
          "duration": 94,
          "from": {
            "location": [
              16.3725,
              48.2085
            ],
            "name": null
          },
          "mode": "walking",
          "summary": "walking",
          "to": {
            "location": [
              16.371607,
              48.208086
            ],
            "name": "Wien Stephansplatz"
          }
        }
      ],
      "summary": "U3",
      "transfers": 0,
      "weight": 865,
      "weight_name": "duration"
    }
  ],
  "timetable": {
    "valid_from": "2025-12-14",
    "valid_to": "2027-12-11"
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Mit dem Rad zur Haltestelle, dann Öffentlicher Verkehr {#bike-to-the-stop-plus-transit}

`GET /v1/directions/bike_transit/{coordinates}`

Wie transit, aber der erste Abschnitt mit dem Rad (Modus `cycling`, bis 15 min). Das Rad bleibt an der Einstiegshaltestelle (es wird nicht mitgenommen), Umstiege und der letzte Abschnitt sind zu Fuß. Kurze Wege können als direkte Radroute kommen.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinates` | path | string | ja | – | lon,lat;lon,lat (genau Start und Ziel) |
| `depart_at` | query | string |  | – | Früheste Abfahrt |
| `arrive_by` | query | string |  | – | Späteste Ankunft |
| `routes` | query | string |  | `3` | 1 bis 5, Standard 3 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/bike_transit/16.3725,48.2085;16.4202,48.1855" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/bike_transit/16.3725,48.2085;16.4202,48.1855", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/bike_transit/16.3725,48.2085;16.4202,48.1855", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "attribution": "mapx.at; Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)",
  "code": "Ok",
  "query": {
    "depart_at": "2026-11-23T08:00:00+01:00",
    "profile": "transit"
  },
  "routes": [
    {
      "arrival": "2026-11-23T08:15:51+01:00",
      "departure": "2026-11-23T08:01:26+01:00",
      "distance": 4801,
      "duration": 865,
      "geometry": {
        "coordinates": [
          [
            16.3725,
            48.2085
          ]
        ],
        "type": "LineString"
      },
      "legs": [
        {
          "arrival": "2026-11-23T08:03:00+01:00",
          "departure": "2026-11-23T08:01:26+01:00",
          "distance": 117,
          "duration": 94,
          "from": {
            "location": [
              16.3725,
              48.2085
            ],
            "name": null
          },
          "mode": "walking",
          "summary": "walking",
          "to": {
            "location": [
              16.371607,
              48.208086
            ],
            "name": "Wien Stephansplatz"
          }
        }
      ],
      "summary": "U3",
      "transfers": 0,
      "weight": 865,
      "weight_name": "duration"
    }
  ],
  "timetable": {
    "valid_from": "2025-12-14",
    "valid_to": "2027-12-11"
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Abfahrtstafel (GTFS-Fahrplan) {#departure-board-scheduled-gtfs-timetable}

`GET /v1/departures`

Nächste Abfahrten an einer Haltestelle (Id aus `/v1/stops`, z. B. `at:43:7226`) oder an allen Haltestellen um eine Koordinate. Die Betriebstage folgen Kalender und Gültigkeitszeitraum des Fahrplans, Fahrten über Mitternacht sind enthalten; an den Tagen der Zeitumstellung stimmen die Zeiten (GTFS-Zeiten zählen ab Mittag minus 12 Stunden).

Je Abfahrt: Zeit, Linie (`name` wie bei transit: `U1`, `R52`, `RJ 754`; dazu `short_name`, `trip_short_name`, Verkehrsmittel, route_type), Fahrtziel (headsign), Ziel, Bahnsteig (null, wenn unbekannt), Betreiber, Haltestelle. Die Zeiten stammen aus dem Soll-Fahrplan: `realtime` ist immer false. `timetable` nennt den Zeitraum des geladenen Fahrplans (`valid_from`, `valid_to`, alle Fahrplanjahre zusammen). Außerhalb davon 422, unbekannte Haltestelle 404, ungültige Zeit 400. Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG, data.mobilitaetsverbuende.at.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `stop` | query | string | ja | – | Haltestellen-Id (IFOPT `at:<state>:<nr>`, alle Steige), Steig-Id wie `at:43:7226:0:2` (nur dieser Steig; `stops[0]` hat dann `platform_id` und `platform`) oder lon,lat |
| `time` | query | string |  | `now` | Startzeit, ISO 8601 (Ortszeit Wien oder mit `Z`/Offset) oder Unix-Sekunden, Standard jetzt; eine Ortszeit in der Lücke der Umstellung auf Sommerzeit rückt in die Sommerzeit vor |
| `limit` | query | string |  | `20` | 1 bis 100, Standard 20 |
| `radius` | query | string |  | `300` | Nur mit lon,lat: Meter, 1 bis 2000, Standard 300 |
| `horizon` | query | string |  | `14400` | Zeitfenster in Sekunden, 60 bis 86400, Standard 14400 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/departures?stop=16.3725,48.2085&limit=5&radius=300", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "attribution": "mapx.at; Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)",
  "departures": [
    {
      "destination": {
        "id": "at:43:7227",
        "name": "Pöchlarn Bahnhof"
      },
      "headsign": "Pöchlarn Bahnhof",
      "line": {
        "long_name": "Wien - Salzburg - Kufstein - Innsbruck - Bludenz - Bregenz",
        "mode": "rail",
        "name": "R52",
        "route_type": 2,
        "short_name": "R52",
        "trip_short_name": "R 2004"
      },
      "operator": "OEBB Personenverkehr AG Kundenservice",
      "platform": "2",
      "realtime": false,
      "scheduled": "2026-11-23T08:00:00+01:00",
      "stop": {
        "id": "at:43:7226",
        "name": "Melk Bahnhof"
      },
      "time": "2026-11-23T08:00:00+01:00",
      "trip_id": "oebb:28.TA.3-R52-A-j26-1.9.H"
    }
  ],
  "realtime": false,
  "stops": [
    {
      "id": "at:43:7226",
      "lines": [
        "671",
        "683"
      ],
      "location": [
        15.331017,
        48.224588
      ],
      "modes": [
        "bus",
        "rail"
      ],
      "name": "Melk Bahnhof"
    }
  ],
  "time": "2026-11-23T08:00:00+01:00",
  "timetable": {
    "valid_from": "2025-12-14",
    "valid_to": "2027-12-11"
  }
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Haltestellen (GTFS-Stationen, nach IFOPT zusammengeführt) {#stops-gtfs-stations-merged-by-ifopt}

`GET /v1/stops`

Haltestellen in einem Rechteck (`bbox`, nach Zahl der Fahrtmuster absteigend) oder um einen Punkt (`near`, die nächste zuerst, mit `distance` in Metern). Je Haltestelle `id`, `name`, `location` [lon,lat], `modes` (rail, bus, tram, subway, ...) und `lines` (Liniennamen wie in `/v1/departures`, Fernzüge als Gattung wie `RJ`, `RJX`, `IC`).

Als lat,lon angegebene Koordinaten ergeben 400.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north in Grad, höchstens 1 Quadratgrad |
| `near` | query | string |  | – | lon,lat, statt bbox |
| `radius` | query | string |  | `500` | Nur mit near: Meter, 1 bis 5000, Standard 500 |
| `limit` | query | string |  | `50` | 1 bis 500, Standard 50 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/stops?near=16.3725,48.2085&radius=300&limit=5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "attribution": "mapx.at; Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; Wegenetz: GIP.at (CC BY 4.0)",
  "stops": [
    {
      "distance": 219,
      "id": "at:43:7226",
      "lines": [
        "671",
        "683"
      ],
      "location": [
        15.331017,
        48.224588
      ],
      "modes": [
        "bus",
        "rail"
      ],
      "name": "Melk Bahnhof"
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

# ÖV-API: Verbindungen und Abfahrten

Source: https://docs.mapx.at/de/api/transit/

> ÖV-API für den öffentlichen Verkehr in Österreich: Verbindungen, Haltestellen und Abfahrtstafeln aus den Fahrplandaten, mit Parametern und Beispielen.

Verbindungen mit öffentlichem Verkehr (zu Fuß oder mit dem Rad zur Haltestelle), Abfahrtstafeln und Haltestellen auf Basis des Fahrplans.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `route` (5/s), `search` (10/s) · Guide: [ÖV-Abfahrten](https://docs.mapx.at/de/guides/transit.md)

## Öffentlicher Verkehr: zu Fuß und mit Öffis (GTFS-Fahrplan, RAPTOR) {#public-transit-walking-plus-transit-gtfs-timetable-raptor}

`GET /v1/directions/transit/{coordinates}`

Genau Start und Ziel. Verbindungen mit Fußwegen auf dem GIP.at-Netz und Fahrten (Linie, Haltestellen, Zeiten). Zeit über `depart_at` (Standard jetzt) oder `arrive_by`, ISO 8601 in Wiener Ortszeit. Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG, data.mobilitaetsverbuende.at.

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
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/transit/16.3725,48.2085;16.4202,48.1855?depart_at=2026-11-09T08:00", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "code": "Ok",
  "routes": [
    {
      "duration": 2400,
      "legs": [
        {
          "mode": "walking",
          "duration": 300
        },
        {
          "mode": "transit",
          "line": "S1",
          "from": "Wien Hbf",
          "to": "Wien Floridsdorf"
        }
      ]
    }
  ],
  "waypoints": []
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

Nächste Abfahrten an einer Haltestelle (Id aus `/v1/stops`, z. B. `at:43:7226`) oder an allen Haltestellen um eine Koordinate. Die Betriebstage folgen Kalender und Gültigkeitszeitraum des Fahrplans, Fahrten über Mitternacht sind enthalten. Je Abfahrt: Zeit, Linie (Name, Verkehrsmittel, route_type), Fahrtziel (headsign), Ziel, Bahnsteig (null, wenn unbekannt), Betreiber, Haltestelle.

Nur Fahrplandaten, keine aktuellen Abweichungen: `realtime` ist immer false. Außerhalb des Gültigkeitszeitraums 422, unbekannte Haltestelle 404. Fahrplandaten: ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG, data.mobilitaetsverbuende.at.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `stop` | query | string | ja | – | Haltestellen-Id (IFOPT `at:<state>:<nr>`, auch Steig-Id `at:43:7226:0:2`) oder lon,lat |
| `time` | query | string |  | `now` | Startzeit, ISO 8601 (Ortszeit Wien) oder Unix-Sekunden, Standard jetzt |
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

**Beispielantwort**

```json
{
  "time": "2026-11-09T08:00:00+01:00",
  "realtime": false,
  "stops": [
    {
      "id": "at:43:7226",
      "name": "Melk Bahnhof"
    }
  ],
  "departures": [
    {
      "time": "2026-11-09T08:04:00+01:00",
      "line": {
        "name": "REX 1",
        "mode": "rail",
        "route_type": 2
      },
      "headsign": "Wien Westbahnhof",
      "platform": "2",
      "stop": "at:43:7226"
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
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Haltestellen (GTFS-Stationen, nach IFOPT zusammengeführt) {#stops-gtfs-stations-merged-by-ifopt}

`GET /v1/stops`

Haltestellen in einem Rechteck (`bbox`, nach Zahl der Fahrtmuster absteigend) oder um einen Punkt (`near`, die nächste zuerst, mit `distance` in Metern). Je Haltestelle `id`, `name`, `location` [lon,lat], `modes` (rail, bus, tram, subway, ...) und `lines`.

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

**Beispielantwort**

```json
{
  "stops": [
    {
      "id": "at:43:7226",
      "name": "Melk Bahnhof",
      "location": [
        15.3338,
        48.2279
      ],
      "modes": [
        "rail",
        "bus"
      ],
      "lines": [
        "REX 1",
        "S50"
      ]
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

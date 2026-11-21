# Routenoptimierung API

Source: https://docs.mapx.at/de/api/optimization/

> Routenoptimierungs-API: beste Reihenfolge für bis zu 25 Stopps und Tourenplanung für mehrere Fahrzeuge, mit Parametern und Beispielanfragen.

Findet die beste Reihenfolge von Stopps (Handlungsreisender) oder plant Touren für mehrere Fahrzeuge mit Kapazitäten und Zeitfenstern.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

## Optimierte Reihenfolge {#optimized-order}

`GET /v1/optimized-trips/{profile}/{coordinates}`

2 bis 25 Koordinaten. Exakt bis 12 Punkte, darüber Nächster-Nachbar-Verfahren mit 2-opt und Or-opt über die Fahrzeitmatrix. Antwort wie bei Directions: `trips[]` mit einer Route, `waypoints[]` in Eingabereihenfolge mit `waypoint_index` (Position in der Tour) und `trips_index`.

Aufwendig:

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (umfährt aktive Sperren aus den EVIS/ASFINAG-Meldungen und verlangsamt Autobahnabschnitte, auf denen Anzeigen ein gesenktes Tempolimit zeigen; keine flächendeckende Verkehrslage), driving-truck (Lkw über 3,5 t; Fahrzeugmaße über weight/height/width/length/truck_type), cycling, cycling-mtb oder walking |
| `coordinates` | path | string | ja | – | lon,lat;lon,lat;... |
| `source` | query | string |  | – | any (Standard) oder first (Punkt 0 ist der Start) |
| `destination` | query | string |  | – | any (Standard) oder last (letzter Punkt ist das Ende) |
| `roundtrip` | query | string |  | – | true (Standard) kehrt zum Start zurück, oder false; true mit destination=last braucht source=first |
| `steps` | query | string |  | – | true für Abbiegehinweise |
| `geometries` | query | string |  | – | geojson\|polyline\|polyline6 |
| `overview` | query | string |  | – | simplified (Standard) \| full \| false |
| `language` | query | string |  | – | de (Standard) oder en |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/optimized-trips/driving/16.3725,48.2085;15.6256,48.2049;14.2858,48.3069?roundtrip=true", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "code": "Ok",
  "trips": [],
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

## Tourenplanung (vereinfachtes Optimization v2) {#tour-planning-simplified-optimization-v2}

`POST /v1/optimization`

Fahrzeuge mit Start, Ende, Kapazität und Zeitfenster, Aufträge (services, ein Ort) und Sendungen (shipments, Abholung vor Zustellung, dasselbe Fahrzeug) mit Dauer, Zeitfenstern und Mengen. Heuristik: Regret-Einfügung, lokale Suche, Ruin and Recreate bis `options.time_limit_ms`.

Braucht einen geheimen Token (`mx_sk_`); öffentliche Tokens erhalten `403 pk-not-allowed`, im Playground daher nicht ausführbar.

Höchstens 10 Fahrzeuge, 50 Aufträge, 60 verschiedene Orte, 256 KB. Zeiten in Sekunden. Aufwendig:

**Request-Body (JSON)**

| Name | Typ | Beschreibung |
|---|---|---|
| `profile` | string | driving (Standard), cycling, cycling-mtb oder walking |

**Beispiel**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/optimization" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"vehicles":[{"id":1,"start":[16.3725,48.2083],"end":[16.3725,48.2083]}],"services":[{"id":1,"location":[15.6256,48.2049],"service":300},{"id":2,"location":[14.2858,48.3069],"service":300}]}' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/optimization", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify({
    "vehicles": [
      {
        "id": 1,
        "start": [
          16.3725,
          48.2083
        ],
        "end": [
          16.3725,
          48.2083
        ]
      }
    ],
    "services": [
      {
        "id": 1,
        "location": [
          15.6256,
          48.2049
        ],
        "service": 300
      },
      {
        "id": 2,
        "location": [
          14.2858,
          48.3069
        ],
        "service": 300
      }
    ]
  }),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/optimization", headers={"Authorization": "Bearer YOUR_TOKEN"}, json={
    "vehicles": [
      {
        "id": 1,
        "start": [
          16.3725,
          48.2083
        ],
        "end": [
          16.3725,
          48.2083
        ]
      }
    ],
    "services": [
      {
        "id": 1,
        "location": [
          15.6256,
          48.2049
        ],
        "service": 300
      },
      {
        "id": 2,
        "location": [
          14.2858,
          48.3069
        ],
        "service": 300
      }
    ]
  })
data = res.json()
```

**Beispielantwort**

```json
{
  "summary": {
    "cost": 9000,
    "unassigned": 0
  },
  "routes": [],
  "unassigned": []
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `405` | Methode nicht erlaubt |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

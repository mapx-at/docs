# Matrix-API: Fahrzeiten und Distanzen

Source: https://docs.mapx.at/de/api/matrix/

> Matrix-API für Österreich: Fahrzeiten und Distanzen zwischen vielen Start- und Zielpunkten in einer Anfrage, mit Parametern und Beispielen.

Dauer (Sekunden) und Distanz (Meter) zwischen allen Koordinatenpaaren oder nur zwischen `sources` und `destinations`. Profile: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

## Fahrzeit- und Distanzmatrix {#travel-time--distance-matrix}

`GET /v1/matrix/{profile}/{coordinates}`

2 bis 25 Koordinaten. `sources`/`destinations` als Indexliste (`0;2`) oder `all`. Zone `heavy`. Sperren: driving, driving-traffic und driving-truck rechnen mit denselben Sperren und Gewichten wie Directions (verbindliche Sperren nie, übrige Vollsperren wie Wintersperren nur als letzter Ausweg, zeitlich bedingte Regelungen nur bei driving-traffic); jeder Eintrag passt daher zu Dauer und Distanz einer Directions-Anfrage zwischen denselben zwei Punkten.

`closures` meldet, ob Sperren berücksichtigt wurden (`applied`, `count`, `binding`, `binding_count`, `fetched`); cycling und walking beachten keine Sperren und haben kein `closures`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (umfährt aktive Sperren aus den EVIS/ASFINAG-Meldungen und verlangsamt Autobahnabschnitte, auf denen Anzeigen ein gesenktes Tempolimit zeigen; keine flächendeckende Verkehrslage), driving-truck (Lkw über 3,5 t; Fahrzeugmaße über weight/height/width/length/truck_type), cycling, cycling-mtb oder walking |
| `coordinates` | path | string | ja | – | lng,lat;lng,lat;... |
| `sources` | query | string |  | – | Indizes, durch Strichpunkt getrennt |
| `destinations` | query | string |  | – | Indizes, durch Strichpunkt getrennt |
| `annotations` | query | string |  | – | duration (Standard), distance oder beides |
| `weight` | query | string |  | `40` | Nur driving-truck: Gesamtgewicht in t (Standard 40, 0.5 bis 60); meidet Gewichtsklassenverbote und Gewichtsbeschränkungen |
| `height` | query | string |  | `4` | Nur driving-truck: Fahrzeughöhe in m (Standard 4, 1.5 bis 5); meidet Höhenbeschränkungen |
| `width` | query | string |  | `2.55` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
| `length` | query | string |  | – | Nur driving-truck: Fahrzeuglänge in m (2 bis 40); Längenbeschränkungen werden nur geprüft, wenn angegeben |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | Nur driving-truck: semi (Sattelzug, Standard), rigid (Lkw ohne Anhänger) oder trailer (Lkw mit Anhänger) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "durations": [
    [
      0,
      2890.4
    ],
    [
      2901.2,
      0
    ]
  ],
  "sources": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ]
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ]
    }
  ],
  "destinations": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ]
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ]
    }
  ],
  "closures": {
    "applied": true,
    "count": 113,
    "binding": true,
    "binding_count": 8,
    "fetched": "2026-11-22T01:39:31Z"
  }
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `durations` | array | Sekunden |
| `distances` | array | Meter |
| `closures` | object | Berücksichtigte Straßensperren (nur driving, driving-traffic, driving-truck; fehlt bei cycling und walking). |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

# Isochrone-API: erreichbare Gebiete

Source: https://docs.mapx.at/de/api/isochrone/

> Isochrone-API für Österreich: Gebiete, die in vorgegebenen Minuten oder Metern mit Auto, Lkw, Rad, zu Fuß oder mit ÖV erreichbar sind, mit Parametern und Beispielen.

Polygone des Gebiets, das von einem Punkt aus in den angegebenen Zeiten oder Distanzen erreichbar ist. Profile: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`, dazu `transit`.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

## Isochronen {#isochrones}

`GET /v1/isochrone/{profile}/{coordinate}`

Erreichbare Gebiete, größtes zuerst; `properties.contour/color/opacity`. Zone `heavy`. Das Profil `transit` rechnet zu Fuß und mit öffentlichem Verkehr (GTFS-Fahrplan, RAPTOR): zu Fuß zu den Haltestellen, Fahrt ab `depart_at`, von jeder erreichten Haltestelle zu Fuß weiter; nur `contours_minutes` (Standard 5,10,15, höchstens 120 min), es gilt der Gültigkeitszeitraum des Fahrplans (sonst 422), `metadata` enthält `depart_at` (Ortszeit mit Offset) und `transit_stops`.

`access=foot|bike|car` (Standard `foot`) legt fest, wie die erste Haltestelle erreicht wird: `bike` = Bike & Ride (mit dem Rad zu jeder Haltestelle, +2 min zum Absperren), `car` = Park & Ride (mit dem Auto, höchstens 25 min, zu Haltestellen mit Bahn, U-Bahn oder Straßenbahn, +3 min zum Parken, und zu ausgewiesenen P+R-Anlagen der GIP, +2 min und ein Fußweg zur Haltestelle); die schnellste Kombination gewinnt, weiter geht es immer zu Fuß. `park=official` beschränkt das Parken auf ausgewiesene P+R/B+R-Anlagen. `metadata` enthält dann zusätzlich `access`, `park`, `access_stops_improved` und `access_sites` (die ersten `sites` Einträge: `source` site|stop, `kind`, `name`, `lng`, `lat`, `arrive_min`, `ready_min`, `stops_won`). Die Radmitnahme in Zügen steht nicht in den Fahrplandaten, daher gibt es kein `egress=bike`. Die anderen Profile lehnen `depart_at`, `access` und `park` ab (400). Sperren: driving, driving-traffic und driving-truck reichen nicht über aktive Vollsperren aus den Verkehrsmeldungen hinaus (dieselben, die Directions umfährt, siehe `/v1/traffic/incidents` mit `closed: true`), auch nicht über Sperren ohne Verbindlichkeitsvermerk wie Wintersperren von Passstraßen: Eine Isochrone hat kein Ziel hinter der Sperre, einen letzten Ausweg gibt es daher nicht; die Straße, auf der der Startpunkt einrastet, bleibt befahrbar. `closures` meldet, ob Sperren berücksichtigt wurden (`applied`, `count`, `binding`, `binding_count`, `fetched`), `closures_avoided` listet die Sperren, an denen die größte Stufe endet (`id`, `text`, `road`, `category`, `binding`), beide auf oberster Ebene wie bei Directions und Matrix.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking oder transit (zu Fuß und mit öffentlichem Verkehr ab `depart_at`) |
| `coordinate` | path | string | ja | – | lng,lat |
| `contours_minutes` | query | string |  | `5,10,15` | Bis zu 4 Werte, durch Komma getrennt (Standard 5,10,15) |
| `contours_meters` | query | string |  | – | Distanzen statt Minuten (nicht für `transit`) |
| `depart_at` | query | string |  | `now` | Nur `transit`: Abfahrt, ISO 8601 in Ortszeit Europe/Vienna ohne Offset (`2026-11-10T08:00`), mit `Z`/Offset oder als Unix-Sekunden; Standard jetzt |
| `access` | query | string |  | – | Nur `transit`: foot (Standard) \| bike (Bike & Ride) \| car (Park & Ride) |
| `park` | query | string |  | `bike every stop` | Nur `transit` mit bike/car: any (Standard: Rad an jeder Haltestelle, Auto an Bahnhaltestellen) \| official (nur ausgewiesene P+R/B+R-Anlagen) |
| `sites` | query | string |  | `10` | Nur `transit` mit bike/car: Zahl der Zugangseinträge in `metadata.access_sites` (Standard 10, höchstens 50) |
| `weight` | query | string |  | `40` | Nur driving-truck: Gesamtgewicht in t (Standard 40, 0.5 bis 60); meidet Gewichtsklassenverbote und Gewichtsbeschränkungen |
| `height` | query | string |  | `4` | Nur driving-truck: Fahrzeughöhe in m (Standard 4, 1.5 bis 5); meidet Höhenbeschränkungen |
| `width` | query | string |  | `2.55` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
| `length` | query | string |  | – | Nur driving-truck: Fahrzeuglänge in m (2 bis 40); Längenbeschränkungen werden nur geprüft, wenn angegeben |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | Nur driving-truck: semi (Sattelzug, Standard), rigid (Lkw ohne Anhänger) oder trailer (Lkw mit Anhänger) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "contour": 15,
        "metric": "time",
        "color": "#bf4040",
        "opacity": 0.33
      },
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
          [
            [
              [
                16.3,
                48.2
              ],
              [
                16.4,
                48.2
              ]
            ]
          ]
        ]
      }
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

# Tempolimits-API: gesetzliche Limits aus GIP.at

Source: https://docs.mapx.at/de/api/speed-limits/

> Tempolimits-API für Österreich: gesetzliche Tempolimits des Straßennetzes in einem Rechteck oder entlang eines Routenkorridors, mit Parametern und Beispielen.

Gesetzliche Tempolimits der Straßenabschnitte als GeoJSON. Limits auf Autobahn-Anzeigen: [Verkehrs-API](https://docs.mapx.at/de/api/traffic.md).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `route` (5/s) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

## Tempolimits (GIP.at, nur gesetzliche) {#speed-limits-gipat-statutory-limits-only}

`GET /v1/speedlimits`

Gesetzliche Tempolimits des Straßennetzes als kompaktes GeoJSON, entweder in einem Rechteck (`bbox`, höchstens 0,08 Quadratgrad, etwa 28 x 28 km) oder in einem Korridor um eine Route (`corridor` als polyline6, z. B. `routes[].geometry` mit `geometries=polyline6`, dazu `buffer` in Metern).

Ein LineString-Objekt je Straßenabschnitt (Link) in Digitalisierungsrichtung mit `limit_kmh`, `direction` (`both`, `forward` oder `backward` bezogen auf die Geometrie) und `id` (GIP-Link-Id); unterscheiden sich die Richtungen, gibt es ein Objekt je Richtung. Geliefert werden nur gesetzliche Tempolimits (5, 10, 20 ... 130 km/h, einschließlich der allgemeinen Limits im Ortsgebiet und auf Freilandstraßen). Abschnitte ohne Wert (viele Autobahnen) und modellierte Geschwindigkeiten wie 53 oder 84 km/h fehlen. Höchstens 60.000 Objekte. Quelle: GIP.at (CC BY 4.0).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat in Grad, höchstens 0,08 Quadratgrad; statt corridor |
| `corridor` | query | string |  | – | Route als polyline6 (höchstens 20.000 Punkte); statt bbox |
| `buffer` | query | string |  | `30` | Nur mit corridor: Meter um die Route, 1 bis 500, Standard 30 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "legal_only": true,
  "attribution": "mapx.at | GIP.at (CC BY 4.0)",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            15.3294,
            48.2272
          ],
          [
            15.3301,
            48.2275
          ]
        ]
      },
      "properties": {
        "limit_kmh": 50,
        "direction": "both",
        "id": 1543486
      }
    }
  ]
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `legal_only` | boolean | immer true: nur gesetzliche Tempolimits sind enthalten |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

# Map-Matching-API: GPS-Spuren auf dem Straßengraphen

Source: https://docs.mapx.at/de/api/map-matching/

> Map-Matching-API für Österreich: ordnet eine GPS-Spur dem GIP.at-Straßengraphen zu und liefert die zugeordnete Route, mit Parametern und Beispielen.

Ordnet aufgezeichnete Positionen dem Straßengraphen (GIP.at) zu und liefert die wahrscheinlichste Route.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

## Map Matching {#map-matching}

`GET /v1/matching/{profile}/{coordinates}`

2 bis 100 GPS-Punkte, auf das Straßennetz eingepasst. Ohne Zeitstempel gilt ein einzelner Punkt weit abseits der Spur (hin und gleich zurück, beide Schenkel über 300 m und zusammen mehr als das Dreifache des direkten Abstands) als Ausreißer: sein Tracepoint ist `null`, das Matching läuft weiter.

Liegen aufeinanderfolgende Punkte mehr als 5 km (mit Zeitstempeln: 300 s) auseinander, teilt sich die Spur in mehrere `matchings`. Lässt sich kein Teil zuordnen, antwortet die API mit 422 (`No match`). Längere Spuren: `POST https://routing.mapx.at/match` mit einem geheimen Token (`mx_sk_…`; öffentliche Tokens erlauben nur GET) und dem JSON-Body `{"coordinates": [[lng, lat], ...], "timestamps": [...], "radius": 25}` (höchstens 512 KB); die Antwort hat ein eigenes Format, nicht das von Mapbox: eine GeoJSON-FeatureCollection mit einem LineString je eingepasstem Abschnitt und `tracepoints` (eingepasster Punkt und Konfidenz je Eingabepunkt, `null`, wenn nicht zugeordnet).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (umfährt aktive Sperren aus den EVIS/ASFINAG-Meldungen und verlangsamt Autobahnabschnitte, auf denen Anzeigen ein gesenktes Tempolimit zeigen; keine flächendeckende Verkehrslage), driving-truck (Lkw über 3,5 t; Fahrzeugmaße über weight/height/width/length/truck_type), cycling, cycling-mtb oder walking |
| `coordinates` | path | string | ja | – | lng,lat;lng,lat;... |
| `timestamps` | query | string |  | – | Unix-Sekunden, durch Strichpunkt getrennt |
| `radiuses` | query | string |  | – | Ein Wert je Punkt, 5 bis 500 m, durch Semikolon getrennt (das Matching sucht Kandidaten im größten Wert); sonst 400 |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "matchings": [],
  "tracepoints": []
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

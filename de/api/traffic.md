# Verkehrs-API: Meldungen und Tempolimits

Source: https://docs.mapx.at/de/api/traffic/

> Verkehrs-API für Österreich: Verkehrsmeldungen, Tempolimits auf Autobahnen, Anzeigen und Verkehrskacheln, mit Parametern und Beispielanfragen.

Verkehrsmeldungen (Unfälle, Baustellen, Sperren), Tempolimits auf Autobahn-Anzeigen und eine Vektorebene für Karten.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `static` (5/s) · Guide: [Verkehr](https://docs.mapx.at/de/guides/traffic.md)

## Tempolimits der Verkehrsbeeinflussungsanlagen auf Autobahnen (GeoJSON) {#speed-limits-of-variable-message-signs-on-motorways-geojson}

`GET /v1/traffic/flow`

Abschnitte mit einem Tempolimit auf Anzeigen unter der freien Fahrgeschwindigkeit, Stufe 0 bis 3 nach Verhältnis. Keine flächendeckende Verkehrslage. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string | ja | – | west,south,east,north in Grad (WGS84) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/flow?bbox=16.0,48.0,16.8,48.5" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/flow?bbox=16.0,48.0,16.8,48.5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/flow?bbox=16.0,48.0,16.8,48.5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3,
            48.1
          ],
          [
            16.31,
            48.11
          ]
        ]
      },
      "properties": {
        "link": 4711,
        "forward": true,
        "speed_kmh": 80,
        "free_kmh": 130,
        "level": 2,
        "travel_time_s": 48,
        "time": "2026-11-08T11:55:00Z"
      }
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
| `503` | Daten vorübergehend nicht verfügbar |

## Verkehrsmeldungen und Verkehrsregelungen (EVIS.AT): Unfälle, Baustellen, Sperren mit Zeitraum {#incident-reports-and-traffic-regulations-evisat-accidents-roadworks-closures-with-time-span}

`GET /v1/traffic/incidents`

Aktive Meldungen (`active=false`: auch beendete und künftige), filterbar nach Gebiet, Straße (`road`, `road_class`), Route, Art und Zeitfenster. Sortiert nach Wichtigkeit (Sperren und verbindliche Meldungen zuerst, dann Autobahnen, Schnellstraßen, Bundes- und Landesstraßen, dann Beginn), seitenweise mit `limit` und `cursor` (`total`, `next` als relative URL oder `null` auf der letzten Seite; `next_cursor` und `next_offset` nur, wenn es eine weitere Seite gibt).

Baustellen auf Autobahnen und Schnellstraßen enthalten Fahrstreifenangaben der ASFINAG (`lanes`). `mandatory=true`: verbindlich (Router: Sperre), Text unverändert anzeigen. Geometrie entlang der GIP-Kanten. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |
| `planned` | query | string |  | – | true: nur geplante, false: nur ungeplante |
| `road` | query | string |  | – | Straßennummer, normalisiert: A1, A 1 und A01 sind gleich; auch S1, B99, L347 |
| `road_class` | query | string |  | – | Durch Komma getrennt: motorway (A), expressway (S), federal (B), state (L) |
| `active` | query | string |  | `with from/to` | false: auch inaktive (Standard mit from/to) |
| `type` | query | string |  | – | Durch Komma getrennt: accident, congestion, construction, lane_restriction, road_closure, road_hazard, miscellaneous |
| `from` | query | string |  | – | Beginn des Zeitfensters (UTC, YYYY-MM-DD oder YYYY-MM-DDTHH:MM[:SS]Z): Meldungen, die danach noch gelten |
| `to` | query | string |  | – | Ende des Zeitfensters (UTC): Meldungen, die davor beginnen |
| `route` | query | string |  | – | Routenkorridor: codierte Polyline (wie geometries=polyline6 oder polyline) oder GeoJSON-LineString; Ergebnis nach Position auf der Route sortiert |
| `route_precision` | query | string |  | – | 6 (Standard) oder 5 |
| `buffer` | query | string |  | `25` | Halbe Korridorbreite in Metern, 1 bis 1000, Standard 25 |
| `sort` | query | string |  | `with route` | importance (Standard): Sperren und verbindliche Meldungen zuerst, dann Straßenklasse A, S, B, L, dann Beginn; route (Standard mit route): Position auf der Route |
| `limit` | query | string |  | `500` | Höchstens so viele Objekte, 0 bis 1000, Standard 500 |
| `cursor` | query | string |  | – | Seitenmarke: next_cursor der vorigen Antwort |
| `offset` | query | string |  | – | Objekte überspringen (Alternative zu cursor) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&limit=5" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&limit=5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&limit=5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "total": 1,
  "next": null,
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3,
            48.19
          ],
          [
            16.31,
            48.2
          ]
        ]
      },
      "properties": {
        "type": "road_closure",
        "impact": "closed",
        "kind": "roadworks",
        "text": "A23: Sperre Richtung Wien",
        "road": "A23",
        "planned": true,
        "closed": true,
        "mandatory": true,
        "source": "evis-planned+asfinag",
        "sources": [
          "evis",
          "asfinag"
        ],
        "lanes": [
          {
            "direction": "aligned",
            "carriageway": "main",
            "closed": [
              "left"
            ],
            "restricted": 1,
            "open": 1,
            "original": 2,
            "contraflow": false,
            "road_closed": false,
            "speed_limit_kmh": 60
          }
        ],
        "start": "2026-11-08T20:00:00Z",
        "end": "2026-11-09T05:00:00Z"
      }
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
| `503` | Daten vorübergehend nicht verfügbar |

## Wie GET, Parameter als JSON-Objekt (für lange Routen) {#like-get-parameters-as-json-object-for-long-routes}

`POST /v1/traffic/incidents`

Body: dieselben Parameter als JSON-Objekt; `route` als Polyline-String oder GeoJSON-LineString, Zahlen (z. B. `route_precision`, `limit`, `buffer`) als JSON-Zahl oder String. Das Gateway nimmt Bodys bis 64 KB an (größer: 413); bei längeren Routen die Linie vereinfachen oder aufteilen.

Braucht einen geheimen Token (`mx_sk_`); öffentliche Tokens erhalten `403 pk-not-allowed`, im Playground daher nicht ausführbar.

**Request-Body (JSON)**

```json
{
  "route": {
    "type": "LineString",
    "coordinates": [
      [
        16.37,
        48.21
      ],
      [
        16.3,
        48.19
      ]
    ]
  },
  "type": "road_closure,construction",
  "limit": 50
}
```

**Beispiel**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/traffic/incidents" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"route":{"type":"LineString","coordinates":[[16.37,48.21],[16.3,48.19]]},"type":"road_closure,construction","limit":50}' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/incidents", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify({
    "route": {
      "type": "LineString",
      "coordinates": [
        [
          16.37,
          48.21
        ],
        [
          16.3,
          48.19
        ]
      ]
    },
    "type": "road_closure,construction",
    "limit": 50
  }),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/traffic/incidents", headers={"Authorization": "Bearer YOUR_TOKEN"}, json={
    "route": {
      "type": "LineString",
      "coordinates": [
        [
          16.37,
          48.21
        ],
        [
          16.3,
          48.19
        ]
      ]
    },
    "type": "road_closure,construction",
    "limit": 50
  })
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "total": 1,
  "next": null,
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3,
            48.19
          ],
          [
            16.31,
            48.2
          ]
        ]
      },
      "properties": {
        "type": "road_closure",
        "impact": "closed",
        "kind": "roadworks",
        "text": "A23: Sperre Richtung Wien",
        "road": "A23",
        "planned": true,
        "closed": true,
        "mandatory": true,
        "source": "evis-planned+asfinag",
        "sources": [
          "evis",
          "asfinag"
        ],
        "lanes": [
          {
            "direction": "aligned",
            "carriageway": "main",
            "closed": [
              "left"
            ],
            "restricted": 1,
            "open": 1,
            "original": 2,
            "contraflow": false,
            "road_closed": false,
            "speed_limit_kmh": 60
          }
        ],
        "start": "2026-11-08T20:00:00Z",
        "end": "2026-11-09T05:00:00Z"
      }
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
| `405` | Methode nicht erlaubt |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Anzeigen der Verkehrsbeeinflussungsanlagen (ASFINAG) mit Position {#variable-message-signs-asfinag-with-position}

`GET /v1/traffic/signs`

Aktueller Zustand: Tempolimit, Piktogramm, Zusatztext. Nur Anlagen mit Position in der Quelle (etwa 40 von rund 5300); die Felder `total_signs` und `located_signs` nennen die Zahlen. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/signs?bbox=16.0,48.0,16.8,48.5" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/signs?bbox=16.0,48.0,16.8,48.5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/signs?bbox=16.0,48.0,16.8,48.5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "total_signs": 5300,
  "located_signs": 40,
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3,
          48.1
        ]
      },
      "properties": {
        "working": true,
        "speed_kmh": 100,
        "pictogram": "speed",
        "text": "",
        "time": "2026-11-08T11:55:00Z"
      }
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
| `503` | Daten vorübergehend nicht verfügbar |

## TileJSON der Verkehrskacheln {#tilejson-of-traffic-tiles}

`GET /v1/traffic/tiles.json`

Vektorkacheln mit den Layern `traffic` (Tempolimits: level, speed, free) und `incidents` (kind, text, road, planned, closed, mandatory).

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/tiles.json" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/tiles.json", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/tiles.json", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "tilejson": "3.0.0",
  "tiles": [
    "https://api.mapx.at/v1/traffic/tiles/{z}/{x}/{y}.pbf"
  ],
  "minzoom": 5,
  "maxzoom": 14
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Verkehrs-Vektorkachel (MVT) {#traffic-vector-tile-mvt}

`GET /v1/traffic/tiles/{z}/{x}/{y}.pbf`

Zoom 5 bis 14. `Cache-Control` 60 Sekunden. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `z` | path | integer | ja | – | 5 bis 14 |
| `x` | path | integer | ja | – | Spalte |
| `y` | path | string | ja | – | Zeile, mit .pbf |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/tiles/10/558/355.pbf" -o result.pbf
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/tiles/10/558/355.pbf", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/tiles/10/558/355.pbf", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.pbf", "wb").write(res.content)
```

**Beispielantwort**: `application/vnd.mapbox-vector-tile` · MVT, gzip, wenn akzeptiert

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Stand der Verkehrsdaten {#state-of-the-traffic-data}

`GET /v1/traffic/status`

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/status" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/status", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/status", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "available": true,
  "incidents": 120,
  "flow_segments": 900,
  "slowed_arcs": 310,
  "closed_arcs": 14
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

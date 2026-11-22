# Verkehrs-API: Meldungen und Tempolimits

Source: https://docs.mapx.at/de/api/traffic/

> Verkehrs-API für Österreich: Verkehrsmeldungen, Tempolimits auf Autobahnen, Anzeigen und Verkehrskacheln, mit Parametern und Beispielanfragen.

Verkehrsmeldungen (Unfälle, Baustellen, Sperren), Tempolimits auf Autobahn-Anzeigen und eine Vektorebene für Karten.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `static` (5/s) · Guide: [Verkehr](https://docs.mapx.at/de/guides/traffic.md)

## Von Wechselverkehrszeichen angezeigte Tempolimits auf Autobahnen (GeoJSON) {#speed-limits-shown-by-variable-message-signs-on-motorways-geojson}

`GET /v1/traffic/sign-limits`

Ein Feature je Autobahn-Link und Fahrtrichtung, auf dem ASFINAG-Wechselverkehrszeichen ein Tempolimit anzeigen; mehrere Zeichen (eines je Fahrstreifen) werden zusammengefasst, es zählt das niedrigste Limit. Das ist keine gemessene Verkehrslage (keine Staudaten).

Eigenschaften: `speed_limit_kmh` (niedrigstes angezeigtes Limit), `normal_speed_kmh` (Tempo, das der Router ohne Zeichen annimmt), `reduction_kmh` (`normal_speed_kmh - speed_limit_kmh`, mindestens 0), `signs` (Kennungen der Zeichen wie in /v1/traffic/signs), `stale` (true: alle Zeichen seit über 7 Tagen unverändert; angezeigt, aber von driving-traffic nicht verwendet), `time` (UTC, wann das Zeichen zuletzt gesetzt wurde), `link` (GIP-Link) und `forward` (Richtung der Link-Geometrie; die Linie verläuft in Fahrtrichtung). `id` = Link und Richtung. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,süd,ost,nord in Grad (WGS84); optional |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/traffic/sign-limits?bbox=16.0,48.0,16.8,48.5" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/traffic/sign-limits?bbox=16.0,48.0,16.8,48.5", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/traffic/sign-limits?bbox=16.0,48.0,16.8,48.5", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "1203445:f",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            13.0223,
            47.8263
          ],
          [
            13.0192,
            47.8256
          ]
        ]
      },
      "properties": {
        "link": 1203445,
        "forward": true,
        "speed_limit_kmh": 80,
        "normal_speed_kmh": 120,
        "reduction_kmh": 40,
        "signs": [
          "AQ_A01_1_291,593_Q#2030057108"
        ],
        "stale": false,
        "time": "2026-11-19T12:40:35Z"
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

Aktive Meldungen (`active=false`: auch beendete und künftige), filterbar nach Gebiet, Straße (`road`, `road_class`), Route, Typ und Zeitfenster. Sortiert nach Bedeutung (Sperren und verbindliche Meldungen zuerst, dann Autobahnen, Schnellstraßen, Bundes- und Landesstraßen, dann Beginn), seitenweise mit `limit` und `cursor` (`total`, `next` als relative URL oder `null` auf der letzten Seite; `next_cursor` und `next_offset` nur, wenn es eine weitere Seite gibt).

Baustellen auf Autobahnen und Schnellstraßen haben Fahrstreifen-Details der ASFINAG (`lanes`). `mandatory=true`: verbindlich (Router: Sperre), Text unverändert anzeigen. Geometrie entlang der GIP-Kanten. Felder: `type` (accident, congestion, construction, lane_restriction, road_closure, road_hazard, miscellaneous) und `impact` (critical: für Pkw gesperrt; major: verbindlich, Fahrstreifensperre oder Unfall; minor: sonst ungeplant; low: sonst geplant) leitet mapx ab. `kind` ist die Kategorie der Quelle auf Deutsch (z. B. Straßensperre, Bauarbeiten, Belagsarbeiten, Instandhaltung), `management` ihr englisches Gegenstück oder die DATEX-II-Maßnahme. `closed` ist nur true, wenn die Straße für Pkw gesperrt ist; ein `road_closure` mit `closed: false` ist nur für andere Verkehrsarten gesperrt (siehe `transport_modes`, z. B. Fußgänger, Fahrräder) oder der Verkehr wird über die Gegenfahrbahn geführt. `road` ist die Straßennummer (A1, S33, B17, L347; aus dem Straßengraphen abgeleitet, wenn die Quelle keine nennt, null bei Gemeindestraßen), `road_section` die Kennung der Quelle für Rampe oder Ast (A13.33.4). `road_name`, `direction` (Text), `regions` (Bundesländer) und `creator` stammen aus EVIS. Zeiten in UTC. Texte sind reiner Text; spitze Klammern der Quelle werden entfernt. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |
| `planned` | query | string |  | – | true: nur geplante, false: nur ungeplante |
| `road` | query | string |  | – | Straßennummer, normalisiert: A1, A 1 und A01 sind gleich; auch S1, B99, L347. Rampen und Äste der Straße inbegriffen (`road_section` A13.33.4 gehört zur A13) |
| `road_class` | query | string |  | – | Durch Komma getrennt: motorway (A), expressway (S), federal (B), state (L) |
| `active` | query | string |  | `true` | true: nur jetzt aktive Meldungen (Standard true; mit from oder to ist der Standard false); false: auch beendete und künftige |
| `type` | query | string |  | – | Durch Komma getrennt: accident, congestion, construction, lane_restriction, road_closure, road_hazard, miscellaneous |
| `from` | query | string |  | – | Beginn des Zeitfensters: Datum YYYY-MM-DD (00:00 UTC) oder RFC-3339-Zeitpunkt mit Z oder Offset (2026-12-01T10:00:00+01:00); Meldungen, die danach noch gelten |
| `to` | query | string |  | – | Ende des Zeitfensters, gleiches Format: Meldungen, die vorher beginnen. Darf nicht vor from liegen (400) |
| `route` | query | string |  | – | Routenkorridor: codierte Polyline (wie geometries=polyline6 oder polyline) oder GeoJSON-LineString; Ergebnis nach Position auf der Route sortiert |
| `route_precision` | query | string |  | – | 6 (Standard) oder 5 |
| `buffer` | query | string |  | `25` | Halbe Korridorbreite in Metern, 1 bis 1000, Standard 25 |
| `sort` | query | string |  | – | importance: Sperren und verbindliche Meldungen zuerst, dann Straßenklasse A, S, B, L, dann Beginn (gilt ohne sort); route: Position auf der Route (gilt ohne sort, wenn route angegeben ist) |
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

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "type": "FeatureCollection",
  "total": 1,
  "next": null,
  "fetched": "2026-11-21T16:05:12Z",
  "features": [
    {
      "type": "Feature",
      "id": "2318923_258221_1",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            11.4993,
            47.0058
          ]
        ]
      },
      "properties": {
        "type": "road_closure",
        "impact": "critical",
        "kind": "Straßensperre",
        "text": "A13 Brenner Autobahn: In Fahrtrichtung von Brenner (I) nach Innsbruck bei Anschlussstelle Brenner Nord Auffahrtsrampe gesperrt, 25.01.2025 08:00 - 31.01.2028 23:30, Rampensperre.",
        "road": "A13",
        "road_section": "A13.33.4",
        "road_name": "Brenner Autobahn",
        "direction": null,
        "regions": [
          "Tirol"
        ],
        "creator": "ASFINAG",
        "transport_modes": [
          "allMotorizedTraffic"
        ],
        "planned": true,
        "closed": true,
        "mandatory": false,
        "both_directions": false,
        "management": "Road closed",
        "start": "2025-01-25T07:00:00Z",
        "end": "2028-01-31T22:30:00Z",
        "updated": "2026-10-21T11:49:09Z",
        "source": "evis-planned+asfinag",
        "sources": [
          "evis"
        ],
        "lanes": [
          {
            "direction": "aligned",
            "carriageway": "slip",
            "closed": [
              "right"
            ],
            "restricted": 1,
            "open": 0,
            "original": 1,
            "contraflow": false,
            "road_closed": true,
            "speed_limit_kmh": null
          }
        ]
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

Body: dieselben Parameter als JSON-Objekt; `route` als Polyline-String oder GeoJSON-LineString, Zahlen (z. B. `route_precision`, `limit`, `buffer`) als JSON-Zahl oder String. Das Gateway nimmt Bodies bis 64 KB an (größer: 413); längere Routen vereinfachen oder aufteilen.

Braucht einen geheimen Token (`mx_sk_`); öffentliche Tokens erhalten `403 pk-not-allowed`, im Playground daher nicht ausführbar.

Seiten: die Antwort enthält `next_cursor` (keine `next`-URL); denselben Body mit `cursor` auf diesen Wert erneut senden.

**Request-Body (JSON)**

| Name | Typ | Beschreibung |
|---|---|---|
| `bbox` |  | Wie der Query-Parameter `bbox` |
| `planned` |  | Wie der Query-Parameter `planned` |
| `road` |  | Wie der Query-Parameter `road` |
| `road_class` |  | Wie der Query-Parameter `road_class` |
| `active` |  | Wie der Query-Parameter `active` |
| `type` |  | Wie der Query-Parameter `type` |
| `from` |  | Wie der Query-Parameter `from` |
| `to` |  | Wie der Query-Parameter `to` |
| `route` |  | Wie der Query-Parameter `route` |
| `route_precision` |  | Wie der Query-Parameter `route_precision` |
| `buffer` |  | Wie der Query-Parameter `buffer` |
| `sort` |  | Wie der Query-Parameter `sort` |
| `limit` |  | Wie der Query-Parameter `limit` |
| `cursor` |  | Wie der Query-Parameter `cursor` |
| `offset` |  | Wie der Query-Parameter `offset` |

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

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "type": "FeatureCollection",
  "total": 1,
  "next": null,
  "fetched": "2026-11-21T16:05:12Z",
  "features": [
    {
      "type": "Feature",
      "id": "2318923_258221_1",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            11.4993,
            47.0058
          ]
        ]
      },
      "properties": {
        "type": "road_closure",
        "impact": "critical",
        "kind": "Straßensperre",
        "text": "A13 Brenner Autobahn: In Fahrtrichtung von Brenner (I) nach Innsbruck bei Anschlussstelle Brenner Nord Auffahrtsrampe gesperrt, 25.01.2025 08:00 - 31.01.2028 23:30, Rampensperre.",
        "road": "A13",
        "road_section": "A13.33.4",
        "road_name": "Brenner Autobahn",
        "direction": null,
        "regions": [
          "Tirol"
        ],
        "creator": "ASFINAG",
        "transport_modes": [
          "allMotorizedTraffic"
        ],
        "planned": true,
        "closed": true,
        "mandatory": false,
        "both_directions": false,
        "management": "Road closed",
        "start": "2025-01-25T07:00:00Z",
        "end": "2028-01-31T22:30:00Z",
        "updated": "2026-10-21T11:49:09Z",
        "source": "evis-planned+asfinag",
        "sources": [
          "evis"
        ],
        "lanes": [
          {
            "direction": "aligned",
            "carriageway": "slip",
            "closed": [
              "right"
            ],
            "restricted": 1,
            "open": 0,
            "original": 1,
            "contraflow": false,
            "road_closed": true,
            "speed_limit_kmh": null
          }
        ]
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

Aktueller Zustand jedes Zeichens (ein Feature je Zeichen, eine Anzeigebrücke hat eines je Fahrstreifen): `speed_kmh` (angezeigtes Limit, null ohne), `pictogram` (DATEX-II-Piktogrammbeschreibung, z. B. maximumSpeedLimitedToTheFigureIndicated, blankVoid), `text` (Zeilen des Zusatztexts, Array, oft leer), `image` (Bild der Anzeige von der ASFINAG), `time` (UTC, zuletzt gesetzt), `stale` (elektronisches Zeichen seit über 7 Tagen unverändert), `working`.

Position aus der ASFINAG-Anlagentabelle `WVZ_TLS_ASFINAG`: `bearing` (Grad, Fahrtrichtung, für die das Zeichen gilt), `carriageway` (main), `lanes` (lane1 = rechter Fahrstreifen, all = ganze Fahrbahn), `lane_count`, `category` (vms, metalSign, roadworkSign, vtp), `position` (feed: aus dem Zustand, table: aus der Anlagentabelle). `total_signs` und `located_signs` nennen die Anzahlen; Zeichen ohne Position fehlen in der Liste. Datenquelle: ASFINAG / EVIS.AT (CC BY 4.0), Straßennetz GIP.at (CC BY 4.0); Quellenangabe erforderlich.

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
  "total_signs": 5330,
  "located_signs": 5330,
  "features": [
    {
      "type": "Feature",
      "id": "AQ_A01_1_291,593_Q#2030057108",
      "geometry": {
        "type": "Point",
        "coordinates": [
          13.0164232,
          47.824955
        ]
      },
      "properties": {
        "working": true,
        "speed_kmh": 80,
        "pictogram": "maximumSpeedLimitedToTheFigureIndicated",
        "text": [],
        "image": "https://maps.asfinag.at/cache/wvz?image=WVZ_A01_1_972_TU2_F3.A_2_26",
        "time": "2026-11-19T12:40:35Z",
        "stale": false,
        "bearing": 251,
        "carriageway": "main",
        "lanes": [
          "lane2"
        ],
        "lane_count": 2,
        "category": "vms",
        "position": "feed"
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

Vektorkacheln mit den Layern `sign_limits` (von Wechselverkehrszeichen angezeigte Tempolimits, keine Verkehrslage: id, speed_limit, normal_speed, reduction, stale; wie in /v1/traffic/sign-limits) und `incidents` (id, type, impact, kind, text, road, start, end, planned, closed, mandatory; wie in /v1/traffic/incidents).

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
  "fetched": "2026-11-22T00:14:22Z",
  "built": "2026-11-22T00:14:36Z",
  "incidents": 783,
  "signs": 5332,
  "located_signs": 27,
  "flow_segments": 27,
  "slowed_arcs": 27,
  "closed_arcs": 41,
  "soft_closed_arcs": 1271,
  "unmatched_incidents": 2,
  "unknown_links": 2
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

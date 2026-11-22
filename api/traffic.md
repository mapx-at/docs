# Traffic API: incidents and speed limits

Source: https://docs.mapx.at/api/traffic/

> Traffic API for Austria: traffic incidents, motorway speed limits, signs and traffic tiles, with parameters and example requests.

Incident reports (accidents, roadworks, closures), speed limits shown on motorway gantries and a vector layer for maps.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s), `static` (5/s) · Guide: [Traffic](https://docs.mapx.at/guides/traffic.md)

## Speed limits shown by variable message signs on motorways (GeoJSON)

`GET /v1/traffic/sign-limits`

One feature per motorway link and direction where ASFINAG signs show a speed limit; several signs (one per lane) are combined, the lowest limit counts. This is not a measured traffic state (no congestion data). Properties: `speed_limit_kmh` (lowest limit shown), `normal_speed_kmh` (speed the router assumes without the sign), `reduction_kmh` (`normal_speed_kmh - speed_limit_kmh`, at least 0), `signs` (sign ids as in /v1/traffic/signs), `stale` (true: all signs unchanged for more than 7 days; shown, but not used by driving-traffic), `time` (UTC, when the sign was last set), `link` (GIP link) and `forward` (direction of the link geometry; the line runs in driving direction).

`id` = link and direction. Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north in degrees (WGS84); optional |

**Example**

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

**Example response**

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Incident reports and traffic regulations (EVIS.AT): accidents, roadworks, closures with time span

`GET /v1/traffic/incidents`

Active reports (`active=false`: also ended and future), filterable by area, road (`road`, `road_class`), route, type and time window. Sorted by importance (closures and binding reports first, then motorways, expressways, federal and state roads, then start), paged with `limit` and `cursor` (`total`, `next` as relative URL or `null` on the last page; `next_cursor` and `next_offset` only when there is a further page).

Roadworks on motorways and expressways carry lane details from ASFINAG (`lanes`). `mandatory=true`: binding (router: closure), show the text unchanged. Geometry along GIP edges. Fields: `type` (accident, congestion, construction, lane_restriction, road_closure, road_hazard, miscellaneous) and `impact` (critical: closed for cars; major: binding, lane restriction or accident; minor: other unplanned; low: other planned) are derived by mapx. `kind` is the category of the source in German (e.g. Straßensperre, Bauarbeiten, Belagsarbeiten, Instandhaltung), `management` its English counterpart or the DATEX II measure. `closed` is true only when the road is closed for cars; a `road_closure` with `closed: false` is closed for other modes only (see `transport_modes`, e.g. pedestrians, cycles) or traffic is led through on the opposite carriageway. `road` is the road number (A1, S33, B17, L347; derived from the road graph if the source has none, null for municipal streets), `road_section` the source id of a ramp or branch (A13.33.4). `road_name`, `direction` (text), `regions` (federal states) and `creator` come from EVIS. Times are UTC. Texts are plain text; angle brackets from the source are removed. Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |
| `planned` | query | string |  | – | true: planned only, false: unplanned only |
| `road` | query | string |  | – | Road number, normalized: A1, A 1 and A01 are the same; also S1, B99, L347. Includes ramps and junction branches of the road (`road_section` A13.33.4 belongs to A13) |
| `road_class` | query | string |  | – | Comma-separated: motorway (A), expressway (S), federal (B), state (L) |
| `active` | query | string |  | `true` | true: only reports active now (default true; with from or to the default is false); false: also ended and future reports |
| `type` | query | string |  | – | Comma-separated: accident, congestion, construction, lane_restriction, road_closure, road_hazard, miscellaneous |
| `from` | query | string |  | – | Window start: date YYYY-MM-DD (00:00 UTC) or RFC 3339 date-time with Z or offset (2026-12-01T10:00:00+01:00); incidents still valid afterwards |
| `to` | query | string |  | – | Window end, same format: incidents starting before. Must not be before from (400) |
| `route` | query | string |  | – | Route corridor: encoded polyline (like geometries=polyline6 or polyline) or GeoJSON LineString; result sorted by position on the route |
| `route_precision` | query | string |  | – | 6 (default) or 5 |
| `buffer` | query | string |  | `25` | Corridor half-width in meters, 1 to 1000, default 25 |
| `sort` | query | string |  | – | importance: closures and binding reports first, then road class A, S, B, L, then start (used when sort is not given); route: position on the route (used when sort is not given and route is set) |
| `limit` | query | string |  | `500` | At most this many features, 0 to 1000, default 500 |
| `cursor` | query | string |  | – | Page marker: next_cursor of the previous response |
| `offset` | query | string |  | – | Skip features (alternative to cursor) |

**Example**

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

**Example response** (shortened: lists show only their first entries)

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
        "kind": "road_closure",
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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Like GET, parameters as JSON object (for long routes)

`POST /v1/traffic/incidents`

Body: the same parameters as a JSON object; `route` as polyline string or GeoJSON LineString, numbers (e.g. `route_precision`, `limit`, `buffer`) as JSON number or string. The gateway accepts bodies up to 64 KB (larger: 413); for longer routes simplify the line or split it.

Needs a secret token (`mx_sk_`); public tokens get `403 pk-not-allowed`, so the playground cannot run it.

Paging: the response has `next_cursor` (no `next` URL); send the same body again with `cursor` set to it.

**Request body (JSON)**

| Name | Type | Description |
|---|---|---|
| `bbox` |  | Like the query parameter `bbox` |
| `planned` |  | Like the query parameter `planned` |
| `road` |  | Like the query parameter `road` |
| `road_class` |  | Like the query parameter `road_class` |
| `active` |  | Like the query parameter `active` |
| `type` |  | Like the query parameter `type` |
| `from` |  | Like the query parameter `from` |
| `to` |  | Like the query parameter `to` |
| `route` |  | Like the query parameter `route` |
| `route_precision` |  | Like the query parameter `route_precision` |
| `buffer` |  | Like the query parameter `buffer` |
| `sort` |  | Like the query parameter `sort` |
| `limit` |  | Like the query parameter `limit` |
| `cursor` |  | Like the query parameter `cursor` |
| `offset` |  | Like the query parameter `offset` |

**Example**

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

**Example response** (shortened: lists show only their first entries)

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
        "kind": "road_closure",
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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `405` | Method not allowed |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Variable message signs (ASFINAG) with position

`GET /v1/traffic/signs`

Current state of each sign (one feature per sign, a gantry has one per lane): `speed_kmh` (limit shown, null without), `pictogram` (DATEX II pictogram description, e.g. maximumSpeedLimitedToTheFigureIndicated, blankVoid), `text` (lines of additional text, array, often empty), `image` (ASFINAG picture of the display), `time` (UTC, last set), `stale` (electronic sign unchanged for more than 7 days), `working`.

Position from the ASFINAG installation table `WVZ_TLS_ASFINAG`: `bearing` (degrees, driving direction the sign applies to), `carriageway` (main), `lanes` (lane1 = right lane, all = whole carriageway), `lane_count`, `category` (vms, metalSign, roadworkSign, vtp), `position` (feed: from the state, table: from the installation table). `total_signs` and `located_signs` give the numbers; signs without position are not listed. Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |

**Example**

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

**Example response**

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## TileJSON of traffic tiles

`GET /v1/traffic/tiles.json`

Vector tiles with layers `sign_limits` (speed limits shown by variable message signs, not a traffic state: id, speed_limit, normal_speed, reduction, stale; as in /v1/traffic/sign-limits) and `incidents` (id, type, impact, kind, text, road, start, end, planned, closed, mandatory; as in /v1/traffic/incidents).

**Example**

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

**Example response**

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Traffic vector tile (MVT)

`GET /v1/traffic/tiles/{z}/{x}/{y}.pbf`

Zoom 5 to 14. `Cache-Control` 60 seconds. Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `z` | path | integer | yes | – | 5 to 14 |
| `x` | path | integer | yes | – | Column |
| `y` | path | string | yes | – | Row, with .pbf |

**Example**

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

**Example response**: `application/vnd.mapbox-vector-tile` · MVT, gzip if accepted

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## State of the traffic data

`GET /v1/traffic/status`

**Example**

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

**Example response**

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

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

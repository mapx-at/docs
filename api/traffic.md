# Traffic API

Traffic incidents, motorway speed limits, signs and traffic tiles.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Incident reports (accidents, roadworks, closures), speed limits shown on motorway gantries and a vector layer for maps.

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s), `static` (5/s) · Guide: [Traffic](/guides/traffic/)

## Speed limits of variable message signs on motorways (GeoJSON)

`GET /v1/traffic/flow`

Edges with a sign speed limit below free-flow speed, level 0 to 3 by ratio. No area-wide traffic state (the paid source is not used). Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string | yes | – | west,south,east,north in degrees (WGS84) |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/traffic/flow?bbox=16.0,48.0,16.8,48.5" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/flow?bbox=16.0,48.0,16.8,48.5");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/traffic/flow?bbox=16.0,48.0,16.8,48.5")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Incident reports and traffic regulations (EVIS.AT): accidents, roadworks, closures with time span

`GET /v1/traffic/incidents`

Active reports (`active=false`: also ended and future), filterable by area, road (`road`, `road_class`), route, type and time window. Sorted by importance (closures and binding reports first, then motorways, expressways, federal and state roads, then start), paged with `limit` and `cursor` (`total`, `next_cursor`, `next` as relative URL, `next_offset`).

<details>
<summary>More</summary>

Roadworks on motorways and expressways carry lane details from ASFINAG (`lanes`). `mandatory=true`: binding (router: closure), show the text unchanged. Geometry along GIP edges. Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |
| `planned` | query | string |  | – | true: planned only, false: unplanned only |
| `road` | query | string |  | – | Road number, normalized: A1, A 1 and A01 are the same; also S1, B99, L347 |
| `road_class` | query | string |  | – | Comma-separated: motorway (A), expressway (S), federal (B), state (L) |
| `active` | query | string |  | `with from/to` | false: also inactive (default with from/to) |
| `type` | query | string |  | – | Comma-separated: accident, congestion, construction, lane_restriction, road_closure, road_hazard, miscellaneous |
| `from` | query | string |  | – | Window start (UTC, YYYY-MM-DD or YYYY-MM-DDTHH:MM[:SS]Z): incidents still valid afterwards |
| `to` | query | string |  | – | Window end (UTC): incidents starting before |
| `route` | query | string |  | – | Route corridor: encoded polyline (like geometries=polyline6 or polyline) or GeoJSON LineString; result sorted by position on the route |
| `route_precision` | query | string |  | – | 6 (default) or 5 |
| `buffer` | query | string |  | `25` | Corridor half-width in meters, 1 to 1000, default 25 |
| `sort` | query | string |  | `with route` | importance (default): closures and binding reports first, then road class A, S, B, L, then start; route (default with route): position on the route |
| `limit` | query | string |  | `500` | At most this many features, 0 to 1000, default 500 |
| `cursor` | query | string |  | – | Page marker: next_cursor of the previous response |
| `offset` | query | string |  | – | Skip features (alternative to cursor) |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&limit=5" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&limit=5");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&limit=5")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "type": "FeatureCollection",
  "total": 1,
  "next_offset": null,
  "next_cursor": null,
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
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Like GET, parameters as JSON object (for long routes)

`POST /v1/traffic/incidents`

Body: the same parameters as a JSON object; `route` as polyline string or GeoJSON LineString.

**Request body (JSON)**

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

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -X POST "https://api.mapx.at/v1/traffic/incidents" \
  -H "Content-Type: application/json" \
  -d '{"route":{"type":"LineString","coordinates":[[16.37,48.21],[16.3,48.19]]},"type":"road_closure,construction","limit":50}' | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/incidents", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
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

</TabItem>
<TabItem label="Python">

```python

res = requests.post("https://api.mapx.at/v1/traffic/incidents", json={
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

</TabItem>
</Tabs>

**Example response**

```json
{
  "type": "FeatureCollection",
  "total": 1,
  "next_offset": null,
  "next_cursor": null,
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
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `405` | Method not allowed |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Variable message signs (ASFINAG) with position

`GET /v1/traffic/signs`

Current state: speed limit, pictogram, additional text. Only installations with a position in the source (about 40 of roughly 5300); the fields `total_signs` and `located_signs` give the numbers. Data source: ASFINAG / EVIS.AT (CC BY 4.0), road network GIP.at (CC BY 4.0); attribution required.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | west,south,east,north; optional |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/traffic/signs?bbox=16.0,48.0,16.8,48.5" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/signs?bbox=16.0,48.0,16.8,48.5");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/traffic/signs?bbox=16.0,48.0,16.8,48.5")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## TileJSON of traffic tiles

`GET /v1/traffic/tiles.json`

Vector tiles with layers `traffic` (speed limits: level, speed, free) and `incidents` (kind, text, road, planned, closed, mandatory).

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/traffic/tiles.json" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/tiles.json");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/traffic/tiles.json")
data = res.json()
```

</TabItem>
</Tabs>

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

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
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

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/traffic/tiles/10/558/355.pbf" -o result.bin
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/tiles/10/558/355.pbf");
const blob = await res.blob();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/traffic/tiles/10/558/355.pbf")
open("result.bin", "wb").write(res.content)
```

</TabItem>
</Tabs>

**Example response**: `application/vnd.mapbox-vector-tile` · MVT, gzip if accepted

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## State of the traffic data

`GET /v1/traffic/status`



**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/traffic/status" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/traffic/status");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/traffic/status")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "available": true,
  "incidents": 120,
  "flow_segments": 900,
  "slowed_arcs": 310,
  "closed_arcs": 14
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `429` | Rate limit exceeded, wait `Retry-After` seconds |

## Playground

<Playground group="traffic" lang="en" />
# Directions API

Routes for car, truck, bike and foot, isochrones, matrices and map matching.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Routing on the Austrian road graph (GIP.at). Profiles: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

Base URL: `https://api.mapx.at` · Rate-limit zone: `route` (5/s), `heavy` (30/min) · Guide: [Navigation](/guides/navigation/)

## Routing

`GET /v1/directions/{profile}/{coordinates}`

Trucks (`driving-truck`, over 3.5 t): vehicle size via `weight`, `height`, `width`, `length`, `truck_type`; roads are blocked per vehicle (weight-class, semi and trailer bans, height/width/weight/length limits). Truck speeds: at most 80 km/h on motorways, 70 on rural roads, 50 in built-up areas.

<details>
<summary>More</summary>

District-wide bans (free-text ordinances) are not routed; affected roads appear as `steps[].notices[]` (`type: truck_restriction`). Responses add `routes[].truck` (the vehicle used) and `routes[].toll.truck_go_km` (kilometers subject to the GO toll; per step `truck_go_m`). 2 to 25 coordinates; legs are computed individually. Car routes (`driving`) contain `routes[].toll` (vignette and special-toll data), `toll_road`, `toll_road_m` and `steps[].intersections[].classes`; toll source ASFINAG, CC BY 4.0. From the GIP, steps additionally carry `ref`, `destinations`, `milestone`, `maxspeed`, `ascent_m`, `descent_m` and (cycling/walking) `grade_pct`, for cycling and cycling-mtb also `bike_infrastructure` (cycleway, lane, shared, none) and `surface` (paved, cobblestone, unpaved, offroad, unknown); `routes[].summary` has ascent, descent, max grade and elevation range. The default `overview=simplified` returns a simplified geometry, `steps=true` adds step geometry and intersections. Closures: `driving` and `driving-traffic` avoid binding full closures from the traffic reports (EVIS/ASFINAG) and list them in `routes[].closures_avoided`; `driving` without active closures along the route is unchanged. Navigation: `steps=true` with `voice_instructions=true` and/or `banner_instructions=true` adds Mapbox-style voice and banner instructions.

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | yes | – | lon,lat;lon,lat;... |
| `steps` | query | string |  | – | true for turn-by-turn steps (with maneuvers and intersections) |
| `geometries` | query | string |  | – | geojson (default) \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (default, simplified route geometry), full (every vertex) or false (no geometry) |
| `language` | query | string |  | – | de (default) or en |
| `alternatives` | query | string |  | – | true (up to 2), false or 0 to 3; only with exactly 2 coordinates. Alternatives are built via laterally offset waypoints and appear as additional `routes[]` |
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (comma-separated; `weather_warnings` adds `routes[].weather_warnings`, the official warnings whose area the route crosses, without geometry): `legs[].annotation` per geometry segment; speed in m/s, maxspeed `[{speed, unit: km/h} \| {unknown: true}]` (driving only), elevation = mean segment height in m |
| `radiuses` | query | string |  | – | Search radius per coordinate in meters, semicolon-separated, or `unlimited` |
| `approaches` | query | string |  | – | unrestricted or curb per coordinate (validated, not yet used by the router) |
| `bearings` | query | string |  | – | degrees,tolerance per coordinate (validated, not yet used by the router) |
| `incidents` | query | string |  | `true for driving-traffic` | true/false: traffic incidents along the leg as `legs[].incidents`; default true for driving-traffic, otherwise false; cars only |
| `exclude` | query | string |  | – | motorway and/or toll, comma-separated (driving, driving-traffic, driving-truck): `motorway` avoids motorways (category A); `toll` avoids all toll roads: vignette-required ASFINAG motorways and expressways (categories A and S) and special-toll sections (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg). ferry is not supported |
| `voice_instructions` | query | string |  | – | true: `steps[].voiceInstructions` (Mapbox form with announcement and ssmlAnnouncement, language per `language`); requires steps=true; driving profiles only |
| `banner_instructions` | query | string |  | – | true: `steps[].bannerInstructions` (Mapbox form, primary/secondary with text and components; destination signage and road number); requires steps=true; driving profiles only |
| `cycling_surface` | query | string |  | – | any (default), cycling only; paved (close unpaved sections completely) is currently unavailable and returns 400 |
| `weight` | query | string |  | `40` | driving-truck only: gross weight in t (default 40, 0.5 to 60); avoids weight-class bans and weight limits |
| `height` | query | string |  | `4` | driving-truck only: vehicle height in m (default 4, 1.5 to 5); avoids height limits |
| `width` | query | string |  | `2` | driving-truck only: vehicle width in m (default 2.55, 1 to 3.5) |
| `length` | query | string |  | – | driving-truck only: vehicle length in m (2 to 40); length limits are only checked when given |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | driving-truck only: semi (articulated lorry, default), rigid (lorry without trailer) or trailer (lorry with trailer) |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "code": "Ok",
  "routes": [
    {
      "weight_name": "routability",
      "weight": 1850.2,
      "distance": 62340.1,
      "duration": 2890.4,
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3725,
            48.2083
          ],
          [
            15.6256,
            48.2049
          ]
        ]
      },
      "legs": [
        {
          "distance": 62340.1,
          "duration": 2890.4,
          "weight": 1850.2,
          "summary": ""
        }
      ]
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Directions with query parameters

`GET /v1/directions`

Same response as `/v1/directions/{profile}/{coordinates}` (preferred); all query parameters of the path form apply.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | query | string |  | `driving` | Default driving |
| `coordinates` | query | string | yes | – | lon,lat;lon,lat;... |
| `steps` | query | string |  | – | true for steps |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (default) \| full \| false |
| `language` | query | string |  | – | de or en |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "code": "Ok",
  "routes": [
    {
      "weight_name": "routability",
      "weight": 1850.2,
      "distance": 62340.1,
      "duration": 2890.4,
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3725,
            48.2083
          ],
          [
            15.6256,
            48.2049
          ]
        ]
      },
      "legs": [
        {
          "distance": 62340.1,
          "duration": 2890.4,
          "weight": 1850.2,
          "summary": ""
        }
      ]
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Isochrones

`GET /v1/isochrone/{profile}/{coordinate}`

Reachable areas, largest first; `properties.contour/color/opacity`. Zone `heavy`. Profile `transit` computes walking plus public transport (GTFS timetable, RAPTOR): walk to stops, ride from `depart_at`, walk on from every stop reached; only `contours_minutes` (default 5,10,15, at most 120 min), the timetable's validity period applies (otherwise 422), `metadata` carries `depart_at`, `transit_stops` and `attribution`.

<details>
<summary>More</summary>

The other profiles reject `depart_at` (400).

</details>

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking or transit (walking plus public transport from `depart_at`); `mapbox/` prefix allowed |
| `coordinate` | path | string | yes | – | lon,lat |
| `contours_minutes` | query | string |  | `5` | Up to 4 values, comma-separated (default 5,10,15) |
| `contours_meters` | query | string |  | – | Distances instead of minutes (not for `transit`) |
| `depart_at` | query | string |  | `now` | Only `transit`: departure, ISO 8601 local time Europe/Vienna without offset (`2026-11-10T08:00`), or with `Z`/offset, or Unix seconds; default now |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15")
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
              ],
              [
                16.4,
                48.25
              ],
              [
  …
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `404` | Not found (name, path or position) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Travel time / distance matrix

`GET /v1/matrix/{profile}/{coordinates}`

2 to 25 coordinates. `sources`/`destinations` as index list (`0;2`) or `all`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | yes | – | lon,lat;lon,lat;... |
| `sources` | query | string |  | – | Indices, semicolon-separated |
| `destinations` | query | string |  | – | Indices, semicolon-separated |
| `annotations` | query | string |  | – | duration (default), distance or both |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "code": "Ok",
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
  …
```

| Field | Type | Description |
|---|---|---|
| `durations` | array | seconds |
| `distances` | array | meters |

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Map matching

`GET /v1/matching/{profile}/{coordinates}`

2 to 100 GPS points snapped to the road network. Longer traces: POST /match at routing.mapx.at.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | yes | – | lon,lat;lon,lat;... |
| `timestamps` | query | string |  | – | Unix seconds, semicolon-separated |
| `radiuses` | query | string |  | – | Meters per point (largest value counts, 5 to 500) |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |

**Example**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110")
data = res.json()
```

</TabItem>
</Tabs>

**Example response**

```json
{
  "code": "Ok",
  "matchings": [],
  "tracepoints": []
}
```

**Errors** (details: [error catalog](/api/errors/))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing or invalid |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

<Playground group="directions" lang="en" />
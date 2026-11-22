# Directions API: routes for car, truck, bike and foot

Source: https://docs.mapx.at/api/directions/

> Directions API for Austria: routes for car, truck, bike and foot on the GIP.at road graph, plus the corridor graph for offline rerouting, with parameters and examples.

Routing on the Austrian road graph (GIP.at). Profiles: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`. Isochrones, matrix, map matching and speed limits have their own pages: [Isochrone](https://docs.mapx.at/api/isochrone.md), [Matrix](https://docs.mapx.at/api/matrix.md), [Map Matching](https://docs.mapx.at/api/map-matching.md), [Speed limits](https://docs.mapx.at/api/speed-limits.md).

Base URL: `https://api.mapx.at` · Rate-limit zone: `heavy` (30/min), `route` (5/s) · Guide: [Navigation](https://docs.mapx.at/guides/navigation.md)

## Corridor graph for offline rerouting

`GET /v1/corridor-graph`

Compact binary subgraph of the road network around a route, for recomputing a route without network (for example in a dead zone after a wrong turn). `polyline` is the route as polyline6 (e.g. `routes[].geometry` with `geometries=polyline6`), `buffer` the radius around it in meters (default 3000, at most 5000).

Needs a secret token (`mx_sk_`); public tokens get `403 pk-not-allowed`, so the playground cannot run it.

The response body is a gzip stream (`application/vnd.mapx.corridor-graph+gzip`, no `Content-Encoding`) of a `MXCG` v1 file: a version header and the graph id of the server graph, nodes (microdegrees), directed arcs with travel time and weight in ms (the same values the router uses, so offline costs equal server costs), simplified link geometry, road name and reference, road class (FRC/FOW), maximum speed with a statutory flag, link flags (roundabout, ramp, toll, urban, motorway) and the turn restrictions between the arcs including derived U-turn bans. Arcs that cannot be driven are left out. Typically 1 to 3 MB per 100 km of route (compressed). Source: GIP.at (CC BY 4.0).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `polyline` | query | string | yes | – | Route as polyline6 (at most 20,000 points) |
| `profile` | query | string |  | – | driving (default), cycling or walking |
| `buffer` | query | string |  | `3000` | Meters around the route, 100 to 5000, default 3000 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/corridor-graph?polyline=_wp~zAofsf%5C_nDouS_pR_af@&buffer=1000" -o result.bin
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/corridor-graph?polyline=_wp~zAofsf%5C_nDouS_pR_af@&buffer=1000", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/corridor-graph?polyline=_wp~zAofsf%5C_nDouS_pR_af@&buffer=1000", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.bin", "wb").write(res.content)
```

**Example response**: `application/vnd.mapx.corridor-graph+gzip` · Corridor graph (gzip of MXCG v1)

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Routing

`GET /v1/directions/{profile}/{coordinates}`

Trucks (`driving-truck`, over 3.5 t): vehicle size via `weight`, `height`, `width`, `length`, `truck_type`; roads are blocked per vehicle (weight-class, semi and trailer bans, height/width/weight/length limits). Truck speeds: at most 80 km/h on motorways, 70 on rural roads, 50 in built-up areas.

District-wide bans (free-text ordinances) are not routed; affected roads appear as `steps[].notices[]` (`type: truck_restriction`). Responses add `routes[].truck` (the vehicle used) and `routes[].toll.truck_go_distance` (meters subject to the GO toll; per step `truck_go_distance`). 2 to 25 coordinates; legs are computed individually. Car routes (`driving`) contain `routes[].toll` (vignette and special-toll data), `toll_road_distance` and `steps[].intersections[].classes`; toll source ASFINAG, CC BY 4.0. From the GIP, steps additionally carry `ref`, `destinations`, `milestone`, `maxspeed`, `ascent`, `descent` (meters) and (cycling/walking) `grade_pct`, for cycling and cycling-mtb also `bike_infrastructure` (cycleway, lane, shared, none) and `surface` (paved, cobblestone, unpaved, offroad, unknown); `routes[].summary` names the two longest roads (string like `legs[].summary`, with `steps=true`), `routes[].elevation_summary` has ascent, descent, max grade and elevation range. The default `overview=simplified` returns a simplified geometry, `steps=true` adds step geometry and intersections. Closures: `driving`, `driving-traffic` and `driving-truck` avoid the active full closures from the traffic reports (EVIS/ASFINAG, `closed: true` in `/v1/traffic/incidents`). Binding closures (EVIS compliance option mandatory, traffic regulations) are never driven; full closures without that flag, such as winter closures of passes, are driven only as a last resort, when start or destination lies on or behind them. `routes[].closures_avoided` lists the closures the route without closures would have used (`binding` tells the two apart), `routes[].closures_unavoidable` those the route still runs through. Time-conditional regulations (weekdays or hours only in the text) are only discouraged, and only by `driving-traffic`. Cycling and walking ignore the closures (the reports concern motor traffic). `driving-traffic` additionally slows motorway sections where ASFINAG variable message signs show a limit below free-flow speed (see `/v1/traffic/sign-limits`); there is no area-wide live traffic state. Navigation: `steps=true` with `voice_instructions=true` and/or `banner_instructions=true` adds Mapbox-style voice and banner instructions (all road profiles, texts in `language`). Speed limits: `maxspeed` (step and annotation) is `{speed, unit, legal}`; `legal: true` marks a statutory limit (5, 10, 20 ... 130 km/h), `legal: false` a modeled GIP speed (e.g. 53, 84) that is not a limit; `{unknown: true}` without a value (backward compatible: `speed` and `unit` are unchanged). For limits along a corridor see `/v1/speedlimits`.

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (avoids active closures from the EVIS/ASFINAG reports and slows motorway sections where variable message signs show a lowered limit; no area-wide live traffic), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking |
| `coordinates` | path | string | yes | – | lng,lat;lng,lat;... |
| `steps` | query | string |  | – | true for turn-by-turn steps (with maneuvers and intersections) |
| `geometries` | query | string |  | – | geojson (default) \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (default, simplified route geometry), full (every vertex) or false (no geometry) |
| `language` | query | string |  | – | de (default) or en for instructions and announcements; other values 400 |
| `alternatives` | query | string |  | – | true (up to 2), false or 0 to 3; only with exactly 2 coordinates. Alternatives are built via laterally offset waypoints and appear as additional `routes[]` |
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (comma-separated; `weather_warnings` adds `routes[].weather_warnings`, the GeoSphere Austria warnings whose area the route crosses, without geometry): `legs[].annotation` per geometry segment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (driving only; `legal` is true for a statutory limit and false for a modeled speed such as 53 or 84 that must not be shown as a limit), elevation = mean segment height in m |
| `radiuses` | query | string |  | – | Search radius per coordinate in meters, semicolon-separated, or `unlimited` (20 km); each coordinate snaps within its own radius, otherwise 404 `no-road-nearby` |
| `approaches` | query | string |  | – | unrestricted or curb per coordinate (validated, not yet used by the router) |
| `bearings` | query | string |  | – | degrees,tolerance per coordinate (validated, not yet used by the router) |
| `incidents` | query | string |  | `true for driving-traffic` | true/false: traffic incidents along the leg as `legs[].incidents`; default true for driving-traffic, otherwise false; cars only |
| `exclude` | query | string |  | – | motorway, toll and/or ferry, comma-separated: `motorway` avoids motorways (category A; driving, driving-traffic, driving-truck); `toll` avoids all toll roads: vignette-required ASFINAG motorways and expressways (categories A and S) and special-toll sections (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg; same profiles). Simplification: the vignette-free motorway sections (e.g. A1 Walserberg to Salzburg Nord, A12 at Kufstein, A14 Hörbranz to Hohenems) are avoided too, because the road graph (GIP) has no vignette attribute; with toll the route reports `toll.vignette_distance` 0, and if no toll-free route exists the request fails instead of falling back to a toll route; `ferry` closes all ferry links (all profiles except driving-traffic, which returns 400). With `ferry` the route is computed without contraction hierarchies, so it takes some tens of milliseconds longer |
| `voice_instructions` | query | string |  | – | true: `steps[].voiceInstructions` (Mapbox form with announcement and ssmlAnnouncement, language per `language`); requires steps=true; all road profiles (driving, driving-truck, cycling, cycling-mtb, walking), with shorter announcement distances on foot (100 m, 40 m) and by bike (250 m, 80 m) |
| `banner_instructions` | query | string |  | – | true: `steps[].bannerInstructions` (Mapbox form, primary/secondary with text and components; destination signage and road number); requires steps=true; all road profiles |
| `cycling_surface` | query | string |  | – | any (default), cycling only; paved (close unpaved sections completely) is currently unavailable and returns 400 |
| `weight` | query | string |  | `40` | driving-truck only: gross weight in t (default 40, 0.5 to 60); avoids weight-class bans and weight limits |
| `height` | query | string |  | `4` | driving-truck only: vehicle height in m (default 4, 1.5 to 5); avoids height limits |
| `width` | query | string |  | `2.55` | driving-truck only: vehicle width in m (default 2.55, 1 to 3.5) |
| `length` | query | string |  | – | driving-truck only: vehicle length in m (2 to 40); length limits are only checked when given |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | driving-truck only: semi (articulated lorry, default), rigid (lorry without trailer) or trailer (lorry with trailer) |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
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
          "summary": "A1, S33"
        }
      ],
      "summary": "A1, S33"
    }
  ],
  "waypoints": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ],
      "distance": 4.2
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ],
      "distance": 6.1
    }
  ],
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
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
| `coordinates` | query | string | yes | – | lng,lat;lng,lat;... |
| `steps` | query | string |  | – | true for turn-by-turn steps (with maneuvers and intersections) |
| `geometries` | query | string |  | – | geojson (default) \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (default, simplified route geometry), full (every vertex) or false (no geometry) |
| `language` | query | string |  | – | de (default) or en for instructions and announcements; other values 400 |
| `alternatives` | query | string |  | – | true (up to 2), false or 0 to 3; only with exactly 2 coordinates. Alternatives are built via laterally offset waypoints and appear as additional `routes[]` |
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (comma-separated; `weather_warnings` adds `routes[].weather_warnings`, the GeoSphere Austria warnings whose area the route crosses, without geometry): `legs[].annotation` per geometry segment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (driving only; `legal` is true for a statutory limit and false for a modeled speed such as 53 or 84 that must not be shown as a limit), elevation = mean segment height in m |
| `radiuses` | query | string |  | – | Search radius per coordinate in meters, semicolon-separated, or `unlimited` (20 km); each coordinate snaps within its own radius, otherwise 404 `no-road-nearby` |
| `approaches` | query | string |  | – | unrestricted or curb per coordinate (validated, not yet used by the router) |
| `bearings` | query | string |  | – | degrees,tolerance per coordinate (validated, not yet used by the router) |
| `incidents` | query | string |  | `true for driving-traffic` | true/false: traffic incidents along the leg as `legs[].incidents`; default true for driving-traffic, otherwise false; cars only |
| `exclude` | query | string |  | – | motorway, toll and/or ferry, comma-separated: `motorway` avoids motorways (category A; driving, driving-traffic, driving-truck); `toll` avoids all toll roads: vignette-required ASFINAG motorways and expressways (categories A and S) and special-toll sections (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg; same profiles). Simplification: the vignette-free motorway sections (e.g. A1 Walserberg to Salzburg Nord, A12 at Kufstein, A14 Hörbranz to Hohenems) are avoided too, because the road graph (GIP) has no vignette attribute; with toll the route reports `toll.vignette_distance` 0, and if no toll-free route exists the request fails instead of falling back to a toll route; `ferry` closes all ferry links (all profiles except driving-traffic, which returns 400). With `ferry` the route is computed without contraction hierarchies, so it takes some tens of milliseconds longer |
| `voice_instructions` | query | string |  | – | true: `steps[].voiceInstructions` (Mapbox form with announcement and ssmlAnnouncement, language per `language`); requires steps=true; all road profiles (driving, driving-truck, cycling, cycling-mtb, walking), with shorter announcement distances on foot (100 m, 40 m) and by bike (250 m, 80 m) |
| `banner_instructions` | query | string |  | – | true: `steps[].bannerInstructions` (Mapbox form, primary/secondary with text and components; destination signage and road number); requires steps=true; all road profiles |
| `cycling_surface` | query | string |  | – | any (default), cycling only; paved (close unpaved sections completely) is currently unavailable and returns 400 |
| `weight` | query | string |  | `40` | driving-truck only: gross weight in t (default 40, 0.5 to 60); avoids weight-class bans and weight limits |
| `height` | query | string |  | `4` | driving-truck only: vehicle height in m (default 4, 1.5 to 5); avoids height limits |
| `width` | query | string |  | `2.55` | driving-truck only: vehicle width in m (default 2.55, 1 to 3.5) |
| `length` | query | string |  | – | driving-truck only: vehicle length in m (2 to 40); length limits are only checked when given |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | driving-truck only: semi (articulated lorry, default), rigid (lorry without trailer) or trailer (lorry with trailer) |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
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
          "summary": "A1, S33"
        }
      ],
      "summary": "A1, S33"
    }
  ],
  "waypoints": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ],
      "distance": 4.2
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ],
      "distance": 6.1
    }
  ],
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Not found (name, path or position) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

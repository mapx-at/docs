# Directions API

Routen für Auto, Lkw, Rad und Fuß, Isochronen, Matrizen und Map Matching.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Routing auf dem österreichischen Straßengraphen (GIP.at). Profile: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `route` (5/s), `heavy` (30/min) · Guide: [Navigation](/de/guides/navigation/)

## Speed limits (GIP.at, statutory limits only)

`GET /v1/speedlimits`

Statutory speed limits of the road network as compact GeoJSON, either inside a box (`bbox`, at most 0.08 square degrees, about 28 x 28 km) or in a corridor around a route (`corridor` as polyline6, e.g. `routes[].geometry` with `geometries=polyline6`, plus `buffer` in meters).

<details>
<summary>Mehr</summary>

One LineString feature per road link in digitizing direction with `limit_kmh`, `direction` (`both`, `forward` or `backward` relative to the geometry) and `id` (GIP link id); if the directions differ there is one feature per direction. Only statutory limits (5, 10, 20 ... 130 km/h, including the default limits inside built-up areas and on rural roads) are returned. Links without a value (many motorways) and modeled speeds such as 53 or 84 km/h are left out. At most 60,000 features. Source: GIP.at (CC BY 4.0).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat in degrees, at most 0.08 square degrees; instead of corridor |
| `corridor` | query | string |  | – | Route as polyline6 (at most 20,000 points); instead of bbox |
| `buffer` | query | string |  | `30` | With corridor only: meters around the route, 1 to 500, default 30 |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23")
data = res.json()
```

</TabItem>
</Tabs>

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "legal_only": true,
  "attribution": "Datenquelle: GIP.at - Graphenintegrations-Plattform, CC BY 4.0",
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
| `legal_only` | boolean | always true: only statutory limits are listed |

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Routing

`GET /v1/directions/{profile}/{coordinates}`

Trucks (`driving-truck`, over 3.5 t): vehicle size via `weight`, `height`, `width`, `length`, `truck_type`; roads are blocked per vehicle (weight-class, semi and trailer bans, height/width/weight/length limits). Truck speeds: at most 80 km/h on motorways, 70 on rural roads, 50 in built-up areas.

<details>
<summary>Mehr</summary>

District-wide bans (free-text ordinances) are not routed; affected roads appear as `steps[].notices[]` (`type: truck_restriction`). Responses add `routes[].truck` (the vehicle used) and `routes[].toll.truck_go_km` (kilometers subject to the GO toll; per step `truck_go_m`). 2 to 25 coordinates; legs are computed individually. Car routes (`driving`) contain `routes[].toll` (vignette and special-toll data), `toll_road`, `toll_road_m` and `steps[].intersections[].classes`; toll source ASFINAG, CC BY 4.0. From the GIP, steps additionally carry `ref`, `destinations`, `milestone`, `maxspeed`, `ascent_m`, `descent_m` and (cycling/walking) `grade_pct`, for cycling and cycling-mtb also `bike_infrastructure` (cycleway, lane, shared, none) and `surface` (paved, cobblestone, unpaved, offroad, unknown); `routes[].summary` has ascent, descent, max grade and elevation range. The default `overview=simplified` returns a simplified geometry, `steps=true` adds step geometry and intersections. Closures: `driving` and `driving-traffic` avoid binding full closures from the traffic reports (EVIS/ASFINAG) and list them in `routes[].closures_avoided`; `driving` without active closures along the route is unchanged. Navigation: `steps=true` with `voice_instructions=true` and/or `banner_instructions=true` adds Mapbox-style voice and banner instructions (all road profiles, texts in `language`). Speed limits: `maxspeed` (step and annotation) is `{speed, unit, legal}`; `legal: true` marks a statutory limit (5, 10, 20 ... 130 km/h), `legal: false` a modeled GIP speed (e.g. 53, 84) that is not a limit; `{unknown: true}` without a value (backward compatible: `speed` and `unit` are unchanged). For limits along a corridor see `/v1/speedlimits`.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | ja | – | lon,lat;lon,lat;... |
| `steps` | query | string |  | – | true for turn-by-turn steps (with maneuvers and intersections) |
| `geometries` | query | string |  | – | geojson (default) \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (default, simplified route geometry), full (every vertex) or false (no geometry) |
| `language` | query | string |  | – | de (default) or en |
| `alternatives` | query | string |  | – | true (up to 2), false or 0 to 3; only with exactly 2 coordinates. Alternatives are built via laterally offset waypoints and appear as additional `routes[]` |
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (comma-separated; `weather_warnings` adds `routes[].weather_warnings`, the official warnings whose area the route crosses, without geometry): `legs[].annotation` per geometry segment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (driving only; `legal` is true for a statutory limit and false for a modeled speed such as 53 or 84 that must not be shown as a limit), elevation = mean segment height in m |
| `radiuses` | query | string |  | – | Search radius per coordinate in meters, semicolon-separated, or `unlimited` |
| `approaches` | query | string |  | – | unrestricted or curb per coordinate (validated, not yet used by the router) |
| `bearings` | query | string |  | – | degrees,tolerance per coordinate (validated, not yet used by the router) |
| `incidents` | query | string |  | `true for driving-traffic` | true/false: traffic incidents along the leg as `legs[].incidents`; default true for driving-traffic, otherwise false; cars only |
| `exclude` | query | string |  | – | motorway, toll and/or ferry, comma-separated: `motorway` avoids motorways (category A; driving, driving-traffic, driving-truck); `toll` avoids all toll roads: vignette-required ASFINAG motorways and expressways (categories A and S) and special-toll sections (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg; same profiles); `ferry` closes all ferry links (all profiles except driving-traffic, which returns 400). With `ferry` the route is computed without contraction hierarchies, so it takes some tens of milliseconds longer |
| `voice_instructions` | query | string |  | – | true: `steps[].voiceInstructions` (Mapbox form with announcement and ssmlAnnouncement, language per `language`); requires steps=true; all road profiles (driving, driving-truck, cycling, cycling-mtb, walking), with shorter announcement distances on foot (100 m, 40 m) and by bike (250 m, 80 m) |
| `banner_instructions` | query | string |  | – | true: `steps[].bannerInstructions` (Mapbox form, primary/secondary with text and components; destination signage and road number); requires steps=true; all road profiles |
| `cycling_surface` | query | string |  | – | any (default), cycling only; paved (close unpaved sections completely) is currently unavailable and returns 400 |
| `weight` | query | string |  | `40` | driving-truck only: gross weight in t (default 40, 0.5 to 60); avoids weight-class bans and weight limits |
| `height` | query | string |  | `4` | driving-truck only: vehicle height in m (default 4, 1.5 to 5); avoids height limits |
| `width` | query | string |  | `2` | driving-truck only: vehicle width in m (default 2.55, 1 to 3.5) |
| `length` | query | string |  | – | driving-truck only: vehicle length in m (2 to 40); length limits are only checked when given |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | driving-truck only: semi (articulated lorry, default), rigid (lorry without trailer) or trailer (lorry with trailer) |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Directions with query parameters

`GET /v1/directions`

Same response as `/v1/directions/{profile}/{coordinates}` (preferred); all query parameters of the path form apply.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | query | string |  | `driving` | Default driving |
| `coordinates` | query | string | ja | – | lon,lat;lon,lat;... |
| `steps` | query | string |  | – | true for steps |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (default) \| full \| false |
| `language` | query | string |  | – | de or en |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Isochrones

`GET /v1/isochrone/{profile}/{coordinate}`

Reachable areas, largest first; `properties.contour/color/opacity`. Zone `heavy`. Profile `transit` computes walking plus public transport (GTFS timetable, RAPTOR): walk to stops, ride from `depart_at`, walk on from every stop reached; only `contours_minutes` (default 5,10,15, at most 120 min), the timetable's validity period applies (otherwise 422), `metadata` carries `depart_at`, `transit_stops` and `attribution`.

<details>
<summary>Mehr</summary>

`access=foot|bike|car` (default `foot`) chooses how the first stop is reached: `bike` = Bike & Ride (bike to any stop, +2 min for locking), `car` = Park & Ride (drive, at most 25 min, to stops with rail/metro/tram service, +3 min for parking, and to official GIP P+R sites, +2 min and a walk to the stop); the fastest combination wins, exit is always on foot. `park=official` restricts parking to official P+R/B+R sites. `metadata` then adds `access`, `park`, `access_stops_improved` and `access_sites` (top `sites` entries: `source` site|stop, `kind`, `name`, `lon`, `lat`, `arrive_min`, `ready_min`, `stops_won`). Bike carriage on trains is not in the timetable data, so there is no `egress=bike`. The other profiles reject `depart_at`, `access` and `park` (400).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking or transit (walking plus public transport from `depart_at`); `mapbox/` prefix allowed |
| `coordinate` | path | string | ja | – | lon,lat |
| `contours_minutes` | query | string |  | `5` | Up to 4 values, comma-separated (default 5,10,15) |
| `contours_meters` | query | string |  | – | Distances instead of minutes (not for `transit`) |
| `depart_at` | query | string |  | `now` | Only `transit`: departure, ISO 8601 local time Europe/Vienna without offset (`2026-11-10T08:00`), or with `Z`/offset, or Unix seconds; default now |
| `access` | query | string |  | – | Only `transit`: foot (default) \| bike (Bike & Ride) \| car (Park & Ride) |
| `park` | query | string |  | `bike every stop` | Only `transit` with bike/car: any (default: bike every stop, car rail stops) \| official (only official P+R/B+R sites) |
| `sites` | query | string |  | `10` | Only `transit` with bike/car: number of access entries in `metadata.access_sites` (default 10, max 50) |

**Beispiel**

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

**Beispielantwort**

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Travel time / distance matrix

`GET /v1/matrix/{profile}/{coordinates}`

2 to 25 coordinates. `sources`/`destinations` as index list (`0;2`) or `all`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | ja | – | lon,lat;lon,lat;... |
| `sources` | query | string |  | – | Indices, semicolon-separated |
| `destinations` | query | string |  | – | Indices, semicolon-separated |
| `annotations` | query | string |  | – | duration (default), distance or both |

**Beispiel**

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

**Beispielantwort**

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

| Feld | Typ | Beschreibung |
|---|---|---|
| `durations` | array | seconds |
| `distances` | array | meters |

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Map matching

`GET /v1/matching/{profile}/{coordinates}`

2 to 100 GPS points snapped to the road network. Longer traces: POST /match at routing.mapx.at.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (with current traffic and closures), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking (`mapbox/` prefix allowed) |
| `coordinates` | path | string | ja | – | lon,lat;lon,lat;... |
| `timestamps` | query | string |  | – | Unix seconds, semicolon-separated |
| `radiuses` | query | string |  | – | Meters per point (largest value counts, 5 to 500) |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |

**Beispiel**

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

**Beispielantwort**

```json
{
  "code": "Ok",
  "matchings": [],
  "tracepoints": []
}
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

<Playground group="directions" lang="de" />
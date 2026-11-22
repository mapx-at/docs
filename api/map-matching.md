# Map Matching API: GPS traces on the road graph

Source: https://docs.mapx.at/api/map-matching/

> Map Matching API for Austria: snaps a GPS trace to the GIP.at road graph and returns the matched route, with parameters and examples.

Snaps recorded positions to the road graph (GIP.at) and returns the most likely route.

Base URL: `https://api.mapx.at` · Rate-limit zone: `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/guides/navigation.md)

## Map matching

`GET /v1/matching/{profile}/{coordinates}`

2 to 100 GPS points snapped to the road network. Without timestamps a single point far off the trace (out and straight back, both legs over 300 m and together more than 3 times the direct distance) counts as an outlier: its tracepoint is `null` and the matching continues.

Consecutive points more than 5 km (or, with timestamps, 300 s) apart split the trace into several `matchings`. If no part can be matched the response is 422 (`No match`). Longer traces: `POST https://routing.mapx.at/match` with a secret token (`mx_sk_…`; public tokens allow GET only) and a JSON body `{"coordinates": [[lng, lat], ...], "timestamps": [...], "radius": 25}` (at most 512 KB); it answers in its own form, not the Mapbox form: a GeoJSON FeatureCollection with one LineString per matched segment plus `tracepoints` (snapped point and confidence per input point, `null` if unmatched).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `profile` | path | string | yes | – | driving, driving-traffic (avoids active closures from the EVIS/ASFINAG reports and slows motorway sections where variable message signs show a lowered limit; no area-wide live traffic), driving-truck (trucks over 3.5 t; vehicle size via weight/height/width/length/truck_type), cycling, cycling-mtb or walking |
| `coordinates` | path | string | yes | – | lng,lat;lng,lat;... |
| `timestamps` | query | string |  | – | Unix seconds, semicolon-separated |
| `radiuses` | query | string |  | – | One value per point, 5 to 500 m, semicolon-separated (the matcher searches candidates within the largest value); otherwise 400 |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |

**Example**

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

**Example response**

```json
{
  "matchings": [],
  "tracepoints": []
}
```

**Errors** (details: [error catalog](https://docs.mapx.at/api/errors.md))

| Status | Meaning |
|---|---|
| `400` | A parameter is missing, invalid or unknown (`unknown-parameter`) |
| `401` | Token missing or invalid (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token not allowed here (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Valid request that cannot be answered (for example no route) |
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `502` | Upstream service unavailable |

## Playground

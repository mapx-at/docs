# Road kilometre API: kilometre posts and coordinates

Source: https://docs.mapx.at/api/road-km/

> Road kilometre API for Austria: position of a kilometre on federal and state roads and the road kilometre at a coordinate, from the kilometre markers of GIP.at.

Turns a road number and kilometre (`B1` km 61.2) into coordinates and a coordinate into the nearest road kilometre. Source: the road kilometre referencing of GIP.at (open government data) following the GIP standard: every hectometre is computed from the kilometred edge sequence and calibrated at the kilometre markers; exactly on a marker you get its position (`kind` `marker`), otherwise a computed point (`kind` `computed`). Motorways and expressways have no markers in GIP.at (rough accuracy), Vienna has no kilometre referencing. The same points are the map layer `road-km`. `/v1/geocode` understands the same input as free text (`B1 km 61,2`).

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](https://docs.mapx.at/guides/search.md)

## Road kilometre to coordinates

`GET /v1/road-km`

Position of a kilometre on a numbered road (A, S, B, L), e.g. `road=B1&km=61.2`. One road can have several sections with the same kilometre (federal states, kilometre jumps, branches); every match is returned, matches closer than 150 m are merged. A kilometre exactly on a marker returns the position of the marker (`exact` true, `kind` `marker`), exactly on a computed hectometre its position (`exact` false, `kind` `computed`).

Otherwise the point is interpolated between the two neighbouring points of the same section (`exact` false, `kind` `computed`, `between` [km, km]): along the GIP.at road geometry (`interpolation` `road`) or, where the geometry does not fit, on the straight line (`interpolation` `straight`, larger `accuracy`); never across a gap (no geometry and a distance more than three times the kilometre difference). Each result: `location` [lng, lat], `state`, `name` (road name or null), `section` (id of the GIP edge sequence), `bearing` (degrees from north in the direction of increasing kilometres), `accuracy` (estimated position uncertainty in meters), `source` (`gip`), `exact`, `kind`, and for interpolated points `between` and `interpolation`. Input is tolerant: `b1`, `B 1`, `B-1`; `km` with point or comma. Unknown road: 404 `unknown-road`; kilometre outside every section (or in a gap): 404 `km-out-of-range` with the extra fields `ranges` [\{state, from, to\}] and `before`/`after` (\{state, km\} of the nearest marker below and above, or null). Source: the road kilometre referencing of GIP.at (open government data, CC BY 4.0) following the GIP standard: every hectometre is computed from the kilometred edge sequence (table LINK_2_EDGE_SEQUENCE) and calibrated at the kilometre markers (table REFERENCE_POINT), the markers themselves keep their position (`kind` `marker`, computed points `kind` `computed`). Motorways and expressways (A, S) have no markers in GIP.at; there the sequence is only roughly calibrated at the junction numbers of the ramps (`accuracy` 300 to 500 m). Vienna has no kilometre referencing in GIP.at. The same points are the tileset `road-km` (`/v1/tiles/road-km`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `road` | query | string | yes | – | Road number: A, S, B or L with 1 to 4 digits and an optional letter, e.g. `B1`, `L100`, `L1234a` |
| `km` | query | string | yes | – | Kilometre, 0 to 2000, point or comma (`61.2`, `61,2`) |
| `state` | query | string |  | – | Only this federal state: name (`Niederösterreich`, also without umlauts, any case), short form (`bgld`, `ktn`, `noe`, `ooe`, `sbg`, `stmk`, `t`, `vbg`, `w`) or ISO code `AT-1` to `AT-9` |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km?road=B1&km=61.2" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km?road=B1&km=61.2", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km?road=B1&km=61.2", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "road": "B1",
  "km": 61.2,
  "results": [
    {
      "location": [
        15.691104,
        48.23548
      ],
      "state": "Niederösterreich",
      "name": "Wiener Straße",
      "section": "gip:244063",
      "bearing": 228,
      "accuracy": 10,
      "source": "gip",
      "exact": false,
      "kind": "computed"
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
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Coordinates to road kilometre

`GET /v1/road-km/reverse`

Nearest roads with kilometre markers around a point: projection onto the road geometry between the two neighbouring markers (straight line where there is none), kilometre in proportion to the length between them, to 10 m (`km` with two decimals); a projection within 1 m of a marker returns the marker (`exact` true, its `km` and position), `distance` in meters from the point to the road, nearest first; per road and federal state only the closest match.

Other fields as in `/v1/road-km`. Empty `results` if no road lies within `radius`. Source: the road kilometre referencing of GIP.at (open government data, CC BY 4.0) following the GIP standard: every hectometre is computed from the kilometred edge sequence (table LINK_2_EDGE_SEQUENCE) and calibrated at the kilometre markers (table REFERENCE_POINT), the markers themselves keep their position (`kind` `marker`, computed points `kind` `computed`). Motorways and expressways (A, S) have no markers in GIP.at; there the sequence is only roughly calibrated at the junction numbers of the ramps (`accuracy` 300 to 500 m). Vienna has no kilometre referencing in GIP.at. The same points are the tileset `road-km` (`/v1/tiles/road-km`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `lng` | query | string | yes | – | Longitude (WGS84) |
| `lat` | query | string | yes | – | Latitude (WGS84) |
| `radius` | query | number (1–500) |  | `50` | Search radius in meters, 1 to 500, default 50 |
| `limit` | query | integer (1–20) |  | `5` | 1 to 20, default 5 |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km/reverse?lng=15.69646&lat=48.23812&radius=50" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km/reverse?lng=15.69646&lat=48.23812&radius=50", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km/reverse?lng=15.69646&lat=48.23812&radius=50", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "location": [
    15.6965,
    48.2381
  ],
  "radius": 50,
  "results": [
    {
      "road": "B1",
      "km": 60.7,
      "location": [
        15.696458,
        48.23812
      ],
      "distance": 0.8,
      "state": "Niederösterreich",
      "name": "Wiener Straße",
      "section": "gip:244063",
      "bearing": 253,
      "accuracy": 5,
      "source": "gip",
      "exact": true,
      "kind": "marker"
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
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Roads with kilometre markers

`GET /v1/road-km/roads`

All roads with kilometre markers and their contiguous kilometre ranges per federal state (`ranges` [\{state, name, from, to\}], `name` is the road name in that state or null; `name` at the top is the first one), sorted by class (A, S, B, L) and number; for autocompletion with `q` (start of the road number, e.g.

`B1` gives B1, B10, B100 ...). Source: the road kilometre referencing of GIP.at (open government data, CC BY 4.0) following the GIP standard: every hectometre is computed from the kilometred edge sequence (table LINK_2_EDGE_SEQUENCE) and calibrated at the kilometre markers (table REFERENCE_POINT), the markers themselves keep their position (`kind` `marker`, computed points `kind` `computed`). Motorways and expressways (A, S) have no markers in GIP.at; there the sequence is only roughly calibrated at the junction numbers of the ramps (`accuracy` 300 to 500 m). Vienna has no kilometre referencing in GIP.at. The same points are the tileset `road-km` (`/v1/tiles/road-km`).

**Parameters**

| Name | In | Type | Required | Default | Description |
|---|---|---|---|---|---|
| `q` | query | string |  | – | Start of the road number, e.g. `B`, `B1`, `L10` (spaces ignored) |
| `state` | query | string |  | – | Only this federal state: name (`Niederösterreich`, also without umlauts, any case), short form (`bgld`, `ktn`, `noe`, `ooe`, `sbg`, `stmk`, `t`, `vbg`, `w`) or ISO code `AT-1` to `AT-9` |

**Example**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/road-km/roads?q=L10&state=Nieder%C3%B6sterreich" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/road-km/roads?q=L10&state=Nieder%C3%B6sterreich", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/road-km/roads?q=L10&state=Nieder%C3%B6sterreich", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Example response**

```json
{
  "roads": [
    {
      "road": "B1",
      "name": "Wiener Straße",
      "ranges": [
        {
          "state": "Niederösterreich",
          "name": "Wiener Straße",
          "from": 16.44,
          "to": 77.2
        },
        {
          "state": "Oberösterreich",
          "name": "Wiener Straße",
          "from": 139.95,
          "to": 300.2
        }
      ]
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
| `429` | Rate limit exceeded, wait `Retry-After` seconds |
| `503` | Data temporarily unavailable |

## Playground

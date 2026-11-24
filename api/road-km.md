# Road kilometre API: kilometre posts and coordinates

Source: https://docs.mapx.at/api/road-km/

> Road kilometre API for Austria: position of a kilometre on federal and state roads and the road kilometre at a coordinate, from the kilometre markers of GIP.at.

Turns a road number and kilometre (`B1` km 61.2) into coordinates and a coordinate into the nearest road kilometre. Source: the kilometre and hectometre markers of GIP.at (open government data) in all federal states; exactly on a marker you get its position, in between the point is interpolated along the road. Motorways, expressways and Vienna have no markers in GIP.at. The same points are the map layer `road-km`. `/v1/geocode` understands the same input as free text (`B1 km 61,2`).

Base URL: `https://api.mapx.at` · Rate-limit zone: `search` (10/s) · Guide: [Search](https://docs.mapx.at/guides/search.md)

## Road kilometre to coordinates

`GET /v1/road-km`

Position of a kilometre on a numbered road (A, S, B, L), e.g. `road=B1&km=61.2`. One road can have several sections with the same kilometre (federal states, kilometre jumps, branches); every match is returned, matches closer than 150 m are merged. A kilometre exactly on a marker returns the position of the marker (`exact` true).

Otherwise the point is interpolated between the two neighbouring markers of the same section (`exact` false, `between` [km, km]): along the GIP.at road geometry (`interpolation` `road`) or, where the geometry does not fit, on the straight line (`interpolation` `straight`, larger `accuracy`); never across a gap (no geometry and a distance more than three times the kilometre difference). Each result: `location` [lng, lat], `state`, `name` (road name or null), `section` (id of the GIP edge sequence), `bearing` (degrees from north in the direction of increasing kilometres), `accuracy` (estimated position uncertainty in meters), `source` (`gip`), `exact`, and for interpolated points `between` and `interpolation`. Input is tolerant: `b1`, `B 1`, `B-1`; `km` with point or comma. Unknown road: 404 `unknown-road`; kilometre outside every section: 404 `km-out-of-range` with the extra field `ranges` [\{state, from, to\}]. Source: the kilometre and hectometre markers of GIP.at (table REFERENCE_POINT, open government data, CC BY 4.0) on federal and state roads of all federal states where GIP.at has them; GIP.at has no markers on motorways and expressways (A, S) and none in Vienna, those are not covered. The same points are the tileset `road-km` (`/v1/tiles/road-km`).

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
      "accuracy": 38,
      "source": "gip",
      "exact": false,
      "between": [
        60.7,
        62.12
      ],
      "interpolation": "road"
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

Other fields as in `/v1/road-km`. Empty `results` if no road lies within `radius`. Source: the kilometre and hectometre markers of GIP.at (table REFERENCE_POINT, open government data, CC BY 4.0) on federal and state roads of all federal states where GIP.at has them; GIP.at has no markers on motorways and expressways (A, S) and none in Vienna, those are not covered. The same points are the tileset `road-km` (`/v1/tiles/road-km`).

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
      "exact": true
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

All roads with kilometre markers and their contiguous kilometre ranges per federal state (`ranges` [\{state, from, to\}]), sorted by class (A, S, B, L) and number; for autocompletion with `q` (start of the road number, e.g. `B1` gives B1, B10, B100 ...). Source: the kilometre and hectometre markers of GIP.at (table REFERENCE_POINT, open government data, CC BY 4.0) on federal and state roads of all federal states where GIP.at has them; GIP.at has no markers on motorways and expressways (A, S) and none in Vienna, those are not covered.

The same points are the tileset `road-km` (`/v1/tiles/road-km`).

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
          "from": 16.44,
          "to": 77.2
        },
        {
          "state": "Oberösterreich",
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

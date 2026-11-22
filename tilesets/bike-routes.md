# Cycle routes tileset (GIP.at)

Source: https://docs.mapx.at/tilesets/bike-routes/

> Signed cycle routes and mountain bike trails in Austria from GIP.at, EuroVelo included, as vector tiles (tileset bike-routes) for MapLibre.

Cycle routes and mountain bike trails from the open data set of the Austrian road graph platform (GIP.at, GIPOGD), EuroVelo routes included. In the style `streets` they form the layer group `bike_routes` (off by default); see [Tourism](https://docs.mapx.at/use-cases/tourism.md).

## Specifications

**Tileset `bike-routes`**

| Property | Value |
|---|---|
| Reference date | `2026-07-08` |
| Format | `mvt` |
| Zoom levels | `z5–z14` |
| Bounds (WGS84) | `9.4980 / 46.2993 / 17.1565 / 49.0141 (W/S/O/N)` |
| License | `CC BY 4.0` |

## Live map

## Add to your map

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/bike-routes.json" } },
    layers: [{ id: "bike_routes", type: "line", source: "v", "source-layer": "bike_routes", paint: { "line-color": "#b5651d", "line-width": 2 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/bike-routes/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/bike-routes.json
```

## Layer reference

### bike_routes

Geometry: **Line**, Zoom: **z5–14**, Layer: `bike_routes`. Signed cycle routes and mountain bike trails, EuroVelo included.

| Field | Type | Description |
|---|---|---|
| `difficulty` | `string` | Difficulty, only for mountain bike trails and not for all of them. Values: `easy` = easy; `medium` = medium; `hard` = hard |
| `min_zoom` | `number` | Zoom level from which the feature is intended to be shown. |
| `name` | `string` | Name of the route. |
| `network` | `string` | Scope of the network. Values: `international` = International route (EuroVelo); `national` = National route; `regional` = Regional route |
| `ref` | `string` | Route number, for example R5 or EV7; absent for routes without a number. |
| `signed` | `boolean` | true if the route is signposted. |
| `type` | `string` | Type of route. Values: `road` = Cycle route on roads and cycle paths; `mtb` = Mountain bike trail |

## Source and currency

- Source: GIP.at (GIPOGD), cycle routes and mountain bike trails. The reference date is in the specifications.

## Notes

- `ref` and `difficulty` are absent for routes without a number or without a rating.

## In other APIs

| | |
|---|---|
| Style | group `bike_routes` in `streets`, off by default |
| Query | [Tilequery](https://docs.mapx.at/api/tilequery.md) with `bike-routes` as tileset |

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from the TileJSON:

```html
mapx.at | Datenquelle: <a href="https://www.gip.gv.at" target="_blank" rel="noopener">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

More: [License](https://docs.mapx.at/reference/attribution.md).

# GIP points tileset (park and ride, level crossings)

Source: https://docs.mapx.at/tilesets/gip-poi/

> Park and ride, bike and ride, level crossings, bollards, barriers, gates and pedestrian crossings in Austria from GIP.at as vector tiles (tileset gip-poi).

Points from the Austrian road graph platform (GIP.at, GIPOGD): park and ride, bike and ride, level crossings, bollards, barriers and gates, and pedestrian crossings. The kind is in `kind`; park and ride and bike and ride appear from z12, all other kinds from z15.

## Specifications

**Tileset `gip-poi`**

| Property | Value |
|---|---|
| Reference date | `2026-07-08` |
| Format | `mvt` |
| Zoom levels | `z12–z15` |
| Bounds (WGS84) | `9.5670 / 46.4991 / 17.0871 / 48.9434 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/gip-poi.json" } },
    layers: [{ id: "gip_poi", type: "circle", source: "v", "source-layer": "gip_poi", paint: { "circle-color": "#2d6cdf", "circle-radius": 5, "circle-stroke-color": "#fff", "circle-stroke-width": 1 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/gip-poi/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/gip-poi.json
```

## Layer reference

### gip_poi

Geometry: **Point**, Zoom: **z12–15**, Layer: `gip_poi`. Point features from GIP.at: park and ride, bike and ride, level crossings, bollards, barriers, gates and pedestrian crossings.

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Type of feature. Values: `park_and_ride` = Park-and-ride site; `park_and_ride_garage` = Park-and-ride garage; `bike_and_ride` = Bike-and-ride site; `level_crossing` = Level crossing; `bollard` = Bollard; `barrier` = Barrier; `gate` = Gate; `crosswalk` = Pedestrian crossing |
| `min_zoom` | `number` | Zoom level from which the feature is intended to be shown: 12 for park and ride and bike and ride, 15 for all others. |
| `name` | `string` | Name of the site for park and ride, bike and ride and level crossings (level crossings: line reference), otherwise absent. |

## Source and currency

- Source: GIP.at (GIPOGD), reference objects, level crossings, nodes and crossings. The reference date is in the specifications.

## Notes

- The open data has no capacities (space counts).
- Park-and-ride and bike-and-ride objects without a position on the road network are missing.

## In other APIs

| | |
|---|---|
| Style | group `gip_poi` in `streets`, off by default |
| Query | [Tilequery](https://docs.mapx.at/api/tilequery.md) with `gip-poi` as tileset |

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from the TileJSON:

```html
mapx.at | Datenquelle: <a href="https://www.gip.gv.at" target="_blank" rel="noopener">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

More: [License](https://docs.mapx.at/reference/attribution.md).

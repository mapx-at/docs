# Rest areas tileset (ASFINAG)

Source: https://docs.mapx.at/tilesets/rest-areas/

> Rest areas and truck parking of ASFINAG as vector tiles (tileset rest-areas) and as the search category rest_area in the mapx search.

Rest, commuter and truck parking sites along motorways and expressways from the ASFINAG traffic data, 314 locations.

> **Warning:** **Attribution**
> 
> mapx.at | Data source: ASFINAG, rest, commuter and truck parking, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Changes: selection and reshaping of fields.

## Specifications

**Tileset `rest-areas`**

| Property | Value |
|---|---|
| Reference date | `2026-11-20` |
| Format | `mvt` |
| Zoom levels | `z7–z14` |
| Bounds (WGS84) | `9.6258 / 46.5314 / 17.1018 / 48.7061 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/rest-areas.json" } },
    layers: [{ id: "rest_area_p", type: "circle", source: "v", "source-layer": "rest_area_p", paint: { "circle-color": "#c0392b", "circle-radius": 5, "circle-stroke-color": "#fff", "circle-stroke-width": 1 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/rest-areas/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/rest-areas.json
```

## Layer reference

### rest_area_p

Geometry: **Point**, Zoom: **z7–14**, Layer: `rest_area_p`. Rest, commuter and truck parking sites of ASFINAG.

| Field | Type | Description |
|---|---|---|
| `car` | `number` | Number of car spaces. |
| `free` | `boolean` | true if free of charge. |
| `id` | `string` | Site identifier (ASFINAG), for example PARK000109. |
| `lorry` | `number` | Number of truck spaces. |
| `min_zoom` | `number` | Zoom level from which the feature is intended to be shown. |
| `name` | `string` | Name of the site. |
| `road` | `string` | Road the site is on, for example A01. |
| `usage` | `string` | Type of site. Values: `truckParking` = Truck parking; `serviceArea` = Service area with facilities; `restArea` = Rest area |

## Source and currency

- Source: ASFINAG traffic data (rest, commuter and truck parking). The reference date is in the specifications.

## Notes

- Occupancy is not included.
- The space counts `car` and `lorry` are the ASFINAG figures; `0` means no spaces of that type.

## In other APIs

| | |
|---|---|
| Style | group `rest_areas`, off by default: point, name from z12; sites with truck spaces darker |
| Search | category `rest_area` in [Search Box](https://docs.mapx.at/api/search-box.md) and geocoding |
| Query | [Tilequery](https://docs.mapx.at/api/tilequery.md) with `rest-areas` as tileset |

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from the TileJSON:

```html
mapx.at | <a href="https://www.asfinag.at" target="_blank" rel="noopener">ASFINAG</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

More: [License](https://docs.mapx.at/reference/attribution.md).

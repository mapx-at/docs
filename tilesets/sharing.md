# Sharing locations tileset (ÖBB)

Source: https://docs.mapx.at/tilesets/sharing/

> Sharing locations (cargo bikes, bikes) of ÖBB as vector tiles (tileset sharing) and as the search category vehicle_sharing in the mapx search API.

Locations of sharing stations from the data set "Shared-Mobility Standorte 2026" of ÖBB-Personenverkehr AG. It only contains sites ÖBB operates and owns itself; currently seven stations in Korneuburg and Leoben.

> **Warning:** **Attribution**
> 
> mapx.at; Data source: ÖBB-Personenverkehr AG, Shared-Mobility Standorte, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de).

## Specifications

**Tileset `sharing`**

| Property | Value |
|---|---|
| Reference date | `2026-11-20` |
| Format | `mvt` |
| Zoom levels | `z8–z14` |
| Bounds (WGS84) | `15.0924 / 47.3621 / 16.3339 / 48.3443 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/sharing.json" } },
    layers: [{ id: "sharing_p", type: "circle", source: "v", "source-layer": "sharing_p", paint: { "circle-color": "#1b7f5a", "circle-radius": 5, "circle-stroke-color": "#fff", "circle-stroke-width": 1 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/sharing/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/sharing.json
```

## Layer reference

### sharing_p

Geometry: **Point**, Zoom: **z8–14**, Layer: `sharing_p`. Sharing locations of ÖBB-Personenverkehr AG.

| Field | Type | Description |
|---|---|---|
| `min_zoom` | `number` | Zoom level from which the feature is intended to be shown. |
| `municipality` | `string` | Municipality. |
| `name` | `string` | Name of the location. |
| `operator` | `string` | Operator, for example ÖBB-PV. |
| `vehicle` | `string` | Vehicle type. Values: `cargo_bike` = Cargo bike; `bike` = Bike |

## Source and currency

- Source: ÖBB-Personenverkehr AG, data set "Shared-Mobility Standorte 2026" (data.oebb.at). The reference date is in the specifications.

## Notes

- The source file has one location of the Leoben station "Waasenbrücke" with a latitude about 40 km off the city; it was not taken over.

## In other APIs

| | |
|---|---|
| Style | group `sharing`, off by default: point from z10, name from z14 |
| Search | category `vehicle_sharing` in [Search Box](https://docs.mapx.at/api/search-box.md) and geocoding |
| Query | [Tilequery](https://docs.mapx.at/api/tilequery.md) with `sharing` as tileset |

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from the TileJSON:

```html
mapx.at; <a href="https://data.oebb.at/de/datensaetze~shared-mobility-standorte~" target="_blank" rel="noopener">ÖBB-Personenverkehr AG – Shared-Mobility Standorte</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

More: [License](https://docs.mapx.at/reference/attribution.md).

# Contour lines tileset (Höhenlinien)

Source: https://docs.mapx.at/tilesets/contours/

> Contour lines for Austria as vector tiles, z10 to z16, 100 m to 10 m interval, derived from the BEV ALS-DTM 1 m.

Contour lines for all of Austria as vector tiles, derived from the [DTM](https://docs.mapx.at/tilesets/terrain-rgb.md). There are four generalization levels, each with its own interval, smoothing and index lines.

**Use cases**

- **Hiking and outdoor maps**: Contours with labeled index lines (https://docs.mapx.at/guides/overlay-contours/)
- **Overlays**: Lines over the basemap or orthophoto (https://docs.mapx.at/guides/overlay-contours/)
- **Route planning**: Read gradient and terrain shape
- **Planning maps**: Elevation bands in thematic maps
- **QGIS**: Vector tiles in desktop GIS (https://docs.mapx.at/guides/qgis/)

## Specifications

**Tileset `hoehenlinien`**

| Property | Value |
|---|---|
| Reference date | `2025-09-15` |
| Format | `mvt` |
| Zoom levels | `z10–z16` |
| Bounds (WGS84) | `9.5311 / 46.3723 / 17.1605 / 49.0210 (W/S/O/N)` |
| License | `CC BY 4.0` |

## Live map

## Add to your map

**MapLibre**

```js
new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" } },
    layers: [{ id: "contour_l", type: "line", source: "v", "source-layer": "contour_l", paint: { "line-color": "#b5651d" } }]
  }
});
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/hoehenlinien-20250915-2037a3df.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](https://docs.mapx.at/guides/self-host-pmtiles/)).

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/contours/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/contours.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/contours/14/8860/5730.pbf
```

## Layer reference

| `lvl` | Zoom | Interval | Index line | Grid | Smoothing (Gaussian σ) | Minimum length |
|---|---|---|---|---|---|---|
| 10 | 10–11 | 100 m | 500 m | 20 m | 60 m | 600 m |
| 12 | 12–13 | 50 m | 200 m | 10 m | 25 m | 250 m |
| 14 | 14–15 | 20 m | 100 m | 4 m | 10 m | 100 m |
| 16 | from 16 | 10 m | 50 m | 2 m | 4 m | 40 m |

There are no lines below z10. Filter by `lvl` in the style, otherwise several levels overlap at z12 and above
if the client overzooms.

```js
{ id: "hl", type: "line", source: "v", "source-layer": "contour_l",
  paint: { "line-color": "#b5651d", "line-width": ["case", ["==", ["get", "index"], 1], 1.4, 0.6] } }
{ id: "hl-text", type: "symbol", source: "v", "source-layer": "contour_l",
  filter: ["==", ["get", "index"], 1],
  layout: { "symbol-placement": "line", "text-field": ["concat", ["get", "ele"], " m"], "text-size": 10 } }
```

### contour_l

Geometry: **Line**, Zoom: **z10–16**, Layer: `contour_l`. Elevation contours in four levels of detail.

| Field | Type | Description |
|---|---|---|
| `ele` | `number` | Elevation of the line in metres above sea level. |
| `index` | `number` | 1 for index contours (major lines), otherwise 0. Values: `0` = Regular contour; `1` = Index contour (major line, usually labelled) |
| `lvl` | `number` | Detail level of the contour; equals the map zoom level from which it is shown (interval 100 m, 50 m, 20 m, 10 m). Values: `10` = Zoom 10–11, 100 m interval (index every 500 m); `12` = Zoom 12–13, 50 m interval (index every 200 m); `14` = Zoom 14–15, 20 m interval (index every 100 m); `16` = Zoom 16, 10 m interval (index every 50 m) |

## Source and currency

- Source: BEV ALS-DTM 1 m, reference date 15 Sep 2025 (same as the DTM).

## Notes

- Because of the smoothing, the line at coarse levels deliberately does not lie exactly on the raw elevation.
- Lines end at the area edge (national border).
- On steep faces 10 m lines are very dense; thinning is the style's job.
- Labeling via the attribute `ele`; there are no dedicated label points.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

```html
Höhenlinien: mapx.at, Datenquelle <a href="https://data.bev.gv.at/geonetwork/srv/ger/catalog.search#/metadata/5ce253fc-b7c5-4362-97af-6556c18a45d9">BEV – ALS-DTM 1 m</a> (Stichtag 15.09.2025), CC BY 4.0
```

More: [License](https://docs.mapx.at/reference/attribution.md).

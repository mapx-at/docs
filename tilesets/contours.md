# Contour lines


Contour lines for all of Austria as vector tiles, derived from the [DTM](/tilesets/terrain-rgb/). There are four generalization levels, each with its own interval, smoothing and index lines.



## Specifications



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
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/hoehenlinien-20250915-2037a3df.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
    **QGIS**

    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/contours/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
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



## Source and currency

- Source: BEV ALS-DTM 1 m, reference date 15 Sep 2025 (same as the DTM).

## Notes

- Because of the smoothing, the line at coarse levels deliberately does not lie exactly on the raw elevation.
- Lines end at the area edge (national border).
- On steep faces 10 m lines are very dense; thinning is the style's job.
- Labeling via the attribute `ele`; there are no dedicated label points.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

{{< attribution ts="hoehenlinien" >}}

More: [License](/reference/attribution/).


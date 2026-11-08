# Base map


The basemap combines open data from BEV, GIP, Umweltbundesamt and BMLUK in 18 layers. It contains no OpenStreetMap data. Cadastre, contour lines and relief are separate products and can be layered on top.



## Specifications



## Live map



The live map shows the layers with automatic colors; your own style defines the final look. Clicking shows the attributes.

## Add to your map

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
        layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
      }
    });
    ```
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/mapx-basis-20260401-6ae3c128.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
    **QGIS**

    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/streets.json
    curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/streets/14/8860/5730.pbf
    ```
  
## Layer reference

A layer name ending in `_a` is an area, `_l` a line, `_p` a point. `min_zoom` on a feature says from which level it is intended; `sort_rank` orders within the layer.



Every field and value of each layer: [Basemap: layer reference](/reference/style-spec/).

## Source and currency

| Topic | Source | As of |
|---|---|---|
| Land use, buildings | BEV Digital Cadastral Map (DKM), heights from the DLM | 1 Apr 2026 |
| Roads, rail | GIP-OGD (Graph Integration Platform) | 30 Apr 2026 |
| Lifts, stations, POI | BEV DLM, Wien-OGD (Vienna only) | |
| Boundaries | BEV administrative boundaries (VGD) | 1 Oct 2026 |
| Places, names, summits | BEV Geographical Names | 23 Mar 2026 |
| House numbers | BEV address register | 1 Oct 2026 |
| Water | BMLUK Gesamtgewässernetz (GGN v19) | |
| Ski areas | Umweltbundesamt | 2017 |
| Airfields | BEV DLM Verkehr (CC BY 4.0) | |


## Notes

- Not included: stops of private railways, subway and tram stops; ski areas as of 2017; small airfields only in part.
- Abroad the map is practically empty (only Lake Constance and waters up to just beyond the border).
- The tileset contains no glyphs or sprites. The mapx styles load them from `https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf` and `/v1/styles/streets/sprite`; a style of your own needs a `glyphs` URL (for example the `/v1/fonts` endpoint, token included).

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

{{< attribution ts="basis" >}}

More: [License](/reference/attribution/).


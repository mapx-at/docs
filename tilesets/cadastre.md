# Cadastre


The boundaries of all parcels as lines, with cadastral municipality number and parcel number, from the BEV Digital Cadastral Map. The tileset has z16 only; clients overzoom above it.



## Specifications



## Live map



## Add to your map

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" } },
        layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
      }
    });
    ```
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/mapx-kataster-20260401-e4504d72.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
    **QGIS**

    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/cadastre/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/cadastre.json
    curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/cadastre/14/8860/5730.pbf
    ```
  
Because only z16 exists, set `minzoom: 16` in the style; the source with `maxzoom: 16` overzooms automatically.

## Layer reference

The lines are boundaries, not areas: a click hits the boundary line and returns the parcel it belongs to. Owner and area are not included.

### parcel_l



## Source and currency

- Source: BEV **Digital Cadastral Map (DKM)**, reference date 1 Apr 2026.

## Notes

> The display is not legally binding. For boundary questions the cadastre at the survey office applies.


- The display is not legally binding; the depiction follows the digital cadastral map.
- There are no tiles below z16.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

{{< attribution ts="kataster" >}}

More: [License](/reference/attribution/).


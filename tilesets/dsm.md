# DSM surface model


The DSM is the counterpart to the [DTM](/tilesets/terrain-rgb/): it shows the surface including buildings, trees and bridges. Encoding, tile size and zoom levels are identical; all clients and formulas of the DTM apply unchanged.



## Specifications



## Live map



Clicking decodes the tile pixel (formula under [Terrain-RGB](/reference/terrain-rgb/)).

## Add to your map

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/dsm.json", tileSize: 512, encoding: "mapbox" } },
        terrain: { source: "dsm", exaggeration: 1.2 },
        layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
      }
    });
    ```
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/relief-dsm-terrainrgb-20250915-d19cb434.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
    **Leaflet**

    ```js
    L.tileLayer("https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN", {
      tileSize: 512, zoomOffset: -1, crossOrigin: true,
      attribution: "BEV, CC BY 4.0"
    }).addTo(map);
    ```
    **QGIS**

    XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN`. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/dsm.json
    curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/dsm/14/8860/5730.webp
    ```
  
Elevation over HTTP: [Elevation and profile](/api/elevation/) with `model=dsm`.

## Source and currency

The DSM is derived from the BEV ALS-DOM 1 m (airborne laser scanning, surface). The reference date is in the specifications and the file name; a file never changes, a new release is a new file.

- Source: BEV ALS-DOM 1 m, reference date 15 Sep 2025.
- Same encoding as the DTM: Mapbox Terrain-RGB with 0.1 m quantization, 512 px, lossless WebP.

## Notes

- Quantization 0.1 m. The finest level is z16 (about 0.8 m per pixel at 48° N); beyond that the client scales up.
- Accuracy corresponds to the official BEV source.
- Buildings and vegetation reflect the time of the survey flight, not the reference date.
- Outside the data the value is a smooth continuation, not a measurement (as with the DTM).
- Building and vegetation height (nDSM = DSM − DTM) can be computed in the client from both tile sets.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

{{< attribution ts="dsm" >}}

More: [License](/reference/attribution/).


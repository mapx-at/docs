# Orthophoto


The BEV orthophoto, re-tiled in Web Mercator up to z18 (20 cm, the native resolution of the source). Which flight supplies a pixel is shown by the [Flight blocks](/tilesets/orthophoto-flight-blocks/) tileset.



## Specifications



## Live map



Clicking shows the flight block at the point (survey, flight period).

## Add to your map

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { ortho: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 } },
        layers: [{ id: "ortho", type: "raster", source: "ortho" }]
      }
    });
    ```
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/dop-ortho-20251003-272a54b3.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
    **Leaflet**

    ```js
    L.tileLayer("https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN", {
      tileSize: 512, zoomOffset: -1, crossOrigin: true,
      attribution: "BEV, CC BY 4.0"
    }).addTo(map);
    ```
    **QGIS**

    XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN`. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto.json
    curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/orthophoto/14/8860/5730.webp
    ```
  
## Source and currency

- Source: BEV **Digital Orthophoto (DOP) RGB**, 20 cm, captured 2022 to 2025.
- Where surveys overlap, the more recent one is shown (no blending).

## Notes

- All of Austria is covered.
- Color seams between surveys are visible; there is no color matching.
- At the borders to DE, CH, LI, IT, SI, CZ and SK the imagery partly extends abroad; at the border to Hungary it ends at the border.
- Below z18, pixels of both surveys mix at block boundaries; the assignment is unambiguous at z18.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

{{< attribution ts="dop" >}}

More: [License](/reference/attribution/).


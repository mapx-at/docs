# Flight blocks


For each pixel of the [orthophoto](/tilesets/orthophoto/), the area of the survey flight (operat) it comes from, with flight period.



## Specifications



## Live map



## Add to your map

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json" } },
        layers: [{ id: "dop_flugblock", type: "line", source: "v", "source-layer": "dop_flugblock", paint: { "line-color": "#b5651d" } }]
      }
    });
    ```
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/dop-flugblock-20251003-0d0d122e.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
    **QGIS**

    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
    curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf
    ```
  
## Layer reference

### dop_flugblock



## Source and currency

- Source: BEV "Currency of orthophoto surveys, color and infrared" (Aktualität Orthophoto Operate Farbe und Infrarot).
- The areas show where a survey is actually visible in the orthophoto. A survey can have several areas.

## Notes

- Date information is the period of the survey, not a daily date.
- A flight block is an operat, not a single flight day: there is no daily date, only the period of the operat; an operat can last up to three months.


## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

{{< attribution ts="dop_flugblock" >}}

More: [License](/reference/attribution/).


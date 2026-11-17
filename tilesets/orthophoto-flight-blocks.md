# Orthophoto flight blocks tileset

Source: https://docs.mapx.at/tilesets/orthophoto-flight-blocks/

> Which flight supplies which orthophoto pixel: winning areas of the BEV surveys with flight period, as vector tiles z6 to z17.

For each pixel of the [orthophoto](https://docs.mapx.at/tilesets/orthophoto.md), the area of the survey flight (operat) it comes from, with flight period.

**Use cases**

- **Show capture date**: "Imagery from" in the map
- **Judge currency**: Age of the aerial image at a location
- **Explain image changes**: Understand color seams between flights
- **Quality checks**: Which flight covers which area

## Specifications

**Tileset `dop-flugblock`**

| Property | Value |
|---|---|
| File | `dop-flugblock-20251003-0d0d122e.pmtiles` |
| Reference date | `2025-10-03` |
| Format | `mvt` |
| Zoom levels | `z6–z17` |
| Bounds (WGS84) | `9.4500 / 46.3000 / 17.2500 / 49.1000 (W/S/O/N)` |
| License | `CC BY 4.0` |

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
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/dop-flugblock-20251003-0d0d122e.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](https://docs.mapx.at/guides/self-host-pmtiles.md)).

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf
```

## Layer reference

### dop_flugblock

Geometry: **Polygon**, Zoom: **z6–17**, Layer: `dop_flugblock`. Orthophoto flight blocks with acquisition date.

| Field | Type | Description |
|---|---|---|
| `archivnr` | `number` | Archive number of the flight. |
| `fid` | `number` | Sequence number of the flight block. |
| `flug_bis` | `string` | End of the flight campaign. |
| `flug_von` | `string` | Start of the flight campaign. |
| `flugblock_id` | `string` | Flight block identifier. |
| `flugdatum` | `string` | Flight date (YYYY-MM-DD). |
| `flugjahr` | `number` | Year of the flight. |
| `mosaik_url` | `string` | Address of the corresponding orthophoto mosaic. |
| `operat` | `string` | Name of the survey project. |
| `rang` | `number` | Priority of the flight block where blocks overlap (0 = highest priority). |
| `stichtag` | `string` | Data cut-off date. |

## Source and currency

- Source: BEV "Currency of orthophoto surveys, color and infrared" (Aktualität Orthophoto Operate Farbe und Infrarot).
- The areas show where a survey is actually visible in the orthophoto. A survey can have several areas.

## Notes

- Date information is the period of the survey, not a daily date.
- A flight block is an operat, not a single flight day: there is no daily date, only the period of the operat; an operat can last up to three months.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

```html
BEV, CC BY 4.0
```

More: [License](https://docs.mapx.at/reference/attribution.md).

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

**Tileset `orthophoto-flight-blocks`**

| Property | Value |
|---|---|
| Reference date | `2025-10-03` |
| Format | `mvt` |
| Zoom levels | `z6–z17` |
| Bounds (WGS84) | `9.4500 / 46.3000 / 17.2500 / 49.1000 (W/S/O/N)` |
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
    sources: { v: { type: "vector", tiles: ["https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf"], minzoom: 6, maxzoom: 17, attribution: "mapx.at | BEV, CC BY 4.0" } },
    layers: [{ id: "orthophoto-flight-blocks", type: "line", source: "v", "source-layer": "flight_block_a", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
The source sets the tile address, zoom range and credit itself; the TileJSON at `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json` provides zoom range and bounds, but only `mapx.at` as credit. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf
```

## Layer reference

### flight_block_a

Geometry: **Polygon**, Zoom: **z6–17**, Layer: `flight_block_a`. Orthophoto flight blocks with acquisition date.

| Field | Type | Description |
|---|---|---|
| `archive_number` | `number` | Archive number of the flight. |
| `block_id` | `string` | Flight block identifier. |
| `fid` | `number` | Sequence number of the flight block. |
| `flight_date` | `string` | End of the flight period (YYYY-MM-DD), decides the priority. |
| `flight_from` | `string` | Start of the flight campaign. |
| `flight_to` | `string` | End of the flight campaign. |
| `flight_year` | `number` | Year of the flight. |
| `mosaic_url` | `string` | Address of the corresponding orthophoto mosaic. |
| `operat` | `string` | Name of the survey project. |
| `rank` | `number` | Priority of the flight block where blocks overlap (0 = highest priority). |
| `reference_date` | `string` | Data cut-off date. |

## Source and currency

- Source: BEV "Currency of orthophoto surveys, color and infrared" (Aktualität Orthophoto Operate Farbe und Infrarot).
- The areas show where a survey is actually visible in the orthophoto. A survey can have several areas.

## Notes

- Date information is the period of the survey, not a daily date.
- A flight block is an operat, not a single flight day: there is no daily date, only the period of the operat; an operat can last up to three months.

## Attribution

Required under CC BY 4.0. The TileJSON only names `mapx.at`; add the source yourself (as in the MapLibre example above):

```html
mapx.at | Orthophoto: BEV (CC BY 4.0)
```

More: [License](https://docs.mapx.at/reference/attribution.md).

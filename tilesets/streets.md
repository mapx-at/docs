# Basemap vector tileset (streets)

Source: https://docs.mapx.at/tilesets/streets/

> Vector base map of Austria from open government data: land use, buildings, roads, rail, water, boundaries, names, house numbers. z0 to z16.

The basemap combines open data from BEV, GIP, Umweltbundesamt and BMLUK in 18 layers. It contains no OpenStreetMap data. Cadastre, contour lines and relief are separate products and can be layered on top.

**Use cases**

- **Custom map style**: Style the layers as you like (https://docs.mapx.at/guides/style-the-basemap/)
- **Web maps and apps**: Vector map with your own design up to z16
- **Planning and navigation background**: Roads, rail, water, boundaries
- **House numbers and names**: Addresses, places, peaks and water names
- **Overlays**: Cadastre, contours and relief on top (https://docs.mapx.at/guides/overlay-contours/)

## Specifications

**Tileset `streets`**

| Property | Value |
|---|---|
| Reference date | `2026-04-01` |
| Format | `mvt` |
| Zoom levels | `z0–z16` |
| Bounds (WGS84) | `8.4375 / 45.0890 / 19.6875 / 50.7365 (W/S/O/N)` |
| License | `CC BY 4.0` |

## Live map

The live map shows the layers with automatic colors; your own style defines the final look. Clicking shows the attributes.

## Add to your map

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
    layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/streets.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/streets/14/8860/5730.pbf
```

## Layer reference

A layer name ending in `_a` is an area, `_l` a line, `_p` a point. `min_zoom` on a feature says from which level it is intended; `sort_rank` orders within the layer.

| Layer | Geometry | Zoom | Fields | Content |
|---|---|---|---|---|

Every field and value of each layer: [Basemap: layer reference](https://docs.mapx.at/reference/style-spec.md).

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

```html
Datenquellen: BEV (DKM, Verwaltungsgrenzen, Adressregister, Geographische Namen), GIP.at, BMLUK (GGN), Umweltbundesamt; CC BY 4.0
```

More: [License](https://docs.mapx.at/reference/attribution.md).

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

**Tileset `mapx-basis`**

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
new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
    layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
  }
});
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options. With PMTiles directly use `url: "pmtiles://https://tiles.mapx.at/mapx-basis-20260401-6ae3c128.pmtiles?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](https://docs.mapx.at/guides/self-host-pmtiles/)).

  

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
| `aerialway_l` | line | z12–16 | `kind` string, `min_zoom` number, `name` string | Aerial ropeways and lifts (DLM): cable_car, gondola, chair_lift, drag_lift … |
| `airport_p` | point | z8–16 | `icao` string, `kind` string, `min_zoom` number, `name` string | Airfields (BEV DLM Verkehr): airport, airfield, heliport, military |
| `boundary_l` | line | z0–16 | `admin_level` number, `kind` string, `left` string, `min_zoom` number, `right` string | Boundaries (BEV VGD): country, state, district, municipality; left/right area on the left/right |
| `boundary_label_p` | point | z4–16 | `admin_level` number, `code` string, `kind` string, `min_zoom` number, `name` string | Names of state, district, municipality; code identifier |
| `building_a` | polygon | z12–16 | `height` number, `kind` string, `max_height` number, `min_zoom` number | Buildings (DKM) with height from the DLM; kind building or church, school, hospital, station, castle, hut …; merged at z12–14 |
| `housenumber_p` | point | z16–16 | `housenumber` string, `kind` string, `street` string | House numbers (BEV address register): kind building = building coordinate, parcel = parcel address |
| `landuse_a` | polygon | z2–16 | `kind` string, `kind_detail` string, `min_zoom` number, `sort_rank` number | Land use from the DKM: residential, garden, farmland, vineyard, alpine_meadow, forest, glacier, rock, water, wetland, transport, quarry, cemetery, recreation |
| `natural_p` | point | z8–16 | `ele` number, `kind` string, `kind_detail` string, `min_zoom` number, `name` string, `sort_rank` number | Mountains, saddles, valleys, field names, castles, ruins … |
| `place_p` | point | z0–16 | `capital` string, `kind` string, `min_zoom` number, `name` string, `name_alt` string, `sort_rank` number | Places (BEV Geographical Names): city, town, village, hamlet, suburb, locality; capital |
| `poi_p` | point | z13–16 | `ele` number, `kind` string, `kind_detail` string, `min_zoom` number, `name` string, `sort_rank` number | Churches, summit crosses and facilities (BEV DLM, in Vienna additionally Wien-OGD): schools, hospitals, huts, petrol stations, museums … |
| `rail_l` | line | z6–16 | `construction` boolean, `kind` string, `kind_detail` string, `min_zoom` number, `structure` string | Rail (GIP): rail, narrow_gauge, rack, subway, tram, funicular, ferry |
| `road_l` | line | z6–16 | `kind` string, `kind_detail` string, `min_zoom` number, `name` string, `oneway` boolean, `ref` string, `sort_rank` number, `structure` string | Roads and paths (GIP): highway, major_road, minor_road, service, track, path; structure bridge/tunnel |
| `ski_a` | polygon | z10–16 | `kind` string, `min_zoom` number | Ski areas (Umweltbundesamt, as of 2017) |
| `transit_p` | point | z10–16 | `kind` string, `min_zoom` number, `name` string | Railway stations, stops, lift stations |
| `water_a` | polygon | z6–16 | `area_km2` number, `kind` string, `min_zoom` number, `name` string, `sort_rank` number | Standing waters (GGN): lake, reservoir, pond |
| `water_l` | line | z6–16 | `catchment_km2` number, `kind` string, `min_zoom` number, `name` string, `sort_rank` number, `structure` string | Flowing waters (GGN): river, stream, canal, ditch |
| `water_label_l` | line | z10–16 | `kind` string, `min_zoom` number, `name` string, `sort_rank` number | Water names along the line |
| `water_label_p` | point | z6–16 | `kind` string, `min_zoom` number, `name` string, `sort_rank` number | Lake names, one point per lake |

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

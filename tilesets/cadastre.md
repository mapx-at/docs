# Cadastre tileset: parcel boundaries

Source: https://docs.mapx.at/tilesets/cadastre/

> Parcel boundaries from the BEV Digital Cadastral Map as vector tiles, z16, with cadastral municipality and parcel number.

The boundaries of all parcels as lines, with cadastral municipality number and parcel number, from the BEV Digital Cadastral Map. The tileset has z16 only; clients overzoom above it.

**Use cases**

- **Parcel search**: Boundaries and numbers on the map
- **Planning and preliminary checks**: Parcel boundaries over the orthophoto
- **Real estate and site maps**: Parcels as a map layer
- **Overlays**: Cadastre over the basemap or aerial imagery (https://docs.mapx.at/guides/style-the-basemap/)
- **QGIS**: Vector tiles in desktop GIS (https://docs.mapx.at/guides/qgis/)

## Specifications

**Tileset `mapx-kataster`**

| Property | Value |
|---|---|
| Reference date | `2026-04-01` |
| Format | `mvt` |
| Zoom levels | `z16–z16` |
| Bounds (WGS84) | `9.5251 / 46.3697 / 17.1661 / 49.0235 (W/S/O/N)` |
| License | `CC BY 4.0` |

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
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**QGIS**

Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/cadastre/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/cadastre.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/cadastre/14/8860/5730.pbf
```

Because only z16 exists, set `minzoom: 16` in the style; the source with `maxzoom: 16` overzooms automatically.

## Layer reference

The lines are boundaries, not areas: a click hits the boundary line and returns the parcel it belongs to. Owner and area are not included.

### parcel_l

Geometry: **Line**, Zoom: **z16–16**, Layer: `parcel_l`. Parcel boundaries (lines).

| Field | Type | Description |
|---|---|---|
| `gstnr` | `string` | Parcel number. |
| `kgnr` | `number` | Cadastral community number. |

## Source and currency

- Source: BEV **Digital Cadastral Map (DKM)**, reference date 1 Apr 2026.

## Notes

> **Warning:** The display is not legally binding. For boundary questions the cadastre at the survey office applies.

- The display is not legally binding; the depiction follows the digital cadastral map.
- There are no tiles below z16.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

```html
Datenquelle: BEV – DKM, CC BY 4.0
```

More: [License](https://docs.mapx.at/reference/attribution.md).

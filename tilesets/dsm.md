# DSM surface model tileset

Source: https://docs.mapx.at/tilesets/dsm/

> Digital surface model of Austria including buildings and vegetation as Terrain-RGB (0.1 m), z5 to z16, from the BEV ALS-DOM 1 m.

The DSM is the counterpart to the [DTM](https://docs.mapx.at/tilesets/terrain-rgb.md): it shows the surface including buildings, trees and bridges. Encoding, tile size and zoom levels are identical; all clients and formulas of the DTM apply unchanged.

**Use cases**

- **Building and vegetation height**: nDSM as DSM minus DTM in the client
- **Solar planning**: Roof and shading heights as a basis
- **Visibility and shading**: Surface including buildings and forest
- **3D city and landscape views**: Surface as terrain in the browser (https://docs.mapx.at/guides/3d-terrain/)
- **Elevation profiles**: Profile over the surface (https://docs.mapx.at/guides/elevation-profile/)

## Specifications

**Tileset `dsm`**

| Property | Value |
|---|---|
| Reference date | `2025-09-15` |
| Format | `webp, encoding mapbox, 512 px` |
| Zoom levels | `z5–z16` |
| Bounds (WGS84) | `9.5306 / 46.3697 / 17.1661 / 49.0235 (W/S/O/N)` |
| Vertical resolution | `0.1 m (Terrain-RGB)` |
| License | `CC BY 4.0` |

## Live map

Clicking decodes the tile pixel (formula under [Terrain-RGB](https://docs.mapx.at/reference/terrain-rgb.md)).

## Add to your map

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/dsm.json", tileSize: 512, encoding: "mapbox" } },
    terrain: { source: "dsm", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**Leaflet**

```js
L.tileLayer("https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "mapx.at; BEV, CC BY 4.0"
}).addTo(map);
```

  

**QGIS**

XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN`.

  

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/dsm.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/dsm/14/8860/5730.webp
```

Elevation over HTTP: [Elevation and profile](https://docs.mapx.at/api/elevation.md) with `model=dsm`.

## Source and currency

The DSM is derived from the BEV ALS-DOM 1 m (airborne laser scanning, surface). The reference date is in the specifications and the file name; a file never changes, a new release is a new file.

- Source: BEV ALS-DOM 1 m, reference date 15 Sep 2025.
- Same encoding as the DTM: Mapbox Terrain-RGB with 0.1 m quantization, 512 px, lossless WebP.

## Notes

- Quantization 0.1 m. The finest level is z16 (about 0.8 m per pixel at 48° N); beyond that the client scales up.
- Accuracy corresponds to the BEV source.
- Buildings and vegetation reflect the time of the survey flight, not the reference date.
- Outside the data the value is a smooth continuation, not a measurement (as with the DTM).
- Building and vegetation height (nDSM = DSM − DTM) can be computed in the client from both tile sets.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from the TileJSON:

```html
mapx.at; Relief: <a href="https://data.bev.gv.at/geonetwork/srv/ger/catalog.search#/metadata/cced2282-cfc5-42eb-9acb-b1c799522fcd">BEV – ALS-DOM 1 m</a> (Stichtag 15.09.2025), CC BY 4.0
```

More: [License](https://docs.mapx.at/reference/attribution.md).

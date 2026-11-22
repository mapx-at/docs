# Orthophoto tileset: aerial imagery

Source: https://docs.mapx.at/tilesets/orthophoto/

> Aerial imagery of Austria from the BEV as WebP tiles, z5 to z18, 20 cm, with an unambiguous flight-block choice per pixel.

The BEV orthophoto, re-tiled in Web Mercator up to z18 (20 cm, the native resolution of the source). Which flight supplies a pixel is shown by the [Flight blocks](https://docs.mapx.at/tilesets/orthophoto-flight-blocks.md) tileset.

**Use cases**

- **Aerial imagery background**: Satellite-map alternative for web and apps
- **3D terrain**: Orthophoto on the DTM (https://docs.mapx.at/guides/3d-terrain/)
- **Planning and documentation**: Situation on site at 20 cm
- **QGIS**: Aerial imagery as XYZ or PMTiles layer (https://docs.mapx.at/guides/qgis/)

## Specifications

**Tileset `orthophoto`**

| Property | Value |
|---|---|
| Reference date | `2025-10-03` |
| Format | `webp` |
| Zoom levels | `z5–z18` |
| Bounds (WGS84) | `9.4500 / 46.3000 / 17.2500 / 49.1000 (W/S/O/N)` |
| License | `CC BY 4.0` |

## Live map

Clicking shows the flight block at the point (survey, flight period).

## Add to your map

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { orthophoto: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 } },
    layers: [{ id: "orthophoto", type: "raster", source: "orthophoto" }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
```
TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](https://docs.mapx.at/start/first-map.md) to the map options.

  

**Leaflet**

```js
L.tileLayer("https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "mapx.at | BEV, CC BY 4.0"
}).addTo(map);
```

  

**QGIS**

XYZ Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=YOUR_TOKEN`.

  

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
- At the borders to DE, CH, LI, IT, SI, CZ and SK the imagery partly extends more than 3 km abroad; at the border to Hungary it ends at the border.
- Below z18, pixels of both surveys mix at block boundaries; the assignment is unambiguous at z18.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from the TileJSON:

```html
mapx.at | Orthophoto: BEV (CC BY 4.0)
```

More: [License](https://docs.mapx.at/reference/attribution.md).

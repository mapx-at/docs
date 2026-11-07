# Show attribution correctly

Credit the sources as CC BY 4.0 requires, in web maps, print and static images: text per product and automatic display in MapLibre.

export const t = data.tilesets;
export const s = (h) => (h || "").replace(/<[^>]+>/g, "");

**Goal:** Show the credit that [CC BY 4.0](/en/lizenz/) requires wherever mapx data is visible.

The obligation: name the source, name the licence (CC BY 4.0), and do not present modifications as official. The credit must be noticeable to users of the map, for example in the corner of the map, not only in an imprint.

## Web map with MapLibre

If you add the tileset via [TileJSON](/en/api/tilejson/) or PMTiles, MapLibre takes the text automatically into the attribution element of the map (bottom right, "i" icon). You need to add nothing as long as you do not hide it:

```js
sources: { dem: { type: "raster-dem", url: "https://tiles.mapx.at/v1/dtm.json" } }
// attribution appears automatically
```

Several sources are merged. If you set `attributionControl: false` or defined a source without TileJSON using `tiles: [...]`, add the text yourself:

```js
{ type: "raster", tiles: ["https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp"], tileSize: 512,
  maxzoom: 18, attribution: "Orthophoto: BEV (CC BY 4.0)" }
```

For Leaflet and OpenLayers, pass the text via `attribution` or `attributions` of the source, see the embedding examples on the [product pages](/en/daten/).

## Print, reports, static images

There is no attribution element there. Put the credit as a visible line in the image or next to the map, in readable size:

```
Source: BEV – ALS-DTM 1 m (cut-off date 15.09.2025), CC BY 4.0
```

If you combine several products, list them all, separated by semicolons. In a report with a map, a line below the figure is enough.

## Text per product

| Product | Text |
|---|---|
| [DTM](/en/daten/dtm/) | {s(t.relief.metadata.attribution)} |
| [DSM](/en/daten/dsm/) | {s(t.dsm.metadata.attribution)} |
| [Orthophoto](/en/daten/orthophoto/) | {s(t.dop.metadata.attribution)} |
| [Basemap](/en/daten/basiskarte/) | {s(t.basis.metadata.attribution)} |
| [Cadastre](/en/daten/kataster/) | {s(t.kataster.metadata.attribution)} |
| [Contour lines](/en/daten/hoehenlinien/) | {s(t.hoehenlinien.metadata.attribution)} |
| [Flight blocks](/en/daten/flugbloecke/) | BEV, CC BY 4.0 |

The texts are also stored in the TileJSON and PMTiles metadata. The version with an HTML link to the source is on the product pages under "Attribution".

<Aside type="tip">
Show the credit in the map itself and additionally in your terms of use. The full licence text is under [License](/en/lizenz/).
</Aside>

## Next steps

- Licence in detail: [License](/en/lizenz/).
- Common questions on licence and commercial use: [FAQ](/en/wissen/faq/).
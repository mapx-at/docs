# Tilesets

All mapx tilesets: type, zoom range and date. One data sheet per tileset.

Each tileset is available as TileJSON and XYZ tiles through the [Tiles API](/api/tiles/) and as one immutable [PMTiles](https://docs.protomaps.com/pmtiles/) file (the source date is part of the file name). All data is licensed CC BY 4.0, see [Attribution](/reference/attribution/).

<TilesetTable />

Use a tileset by name:

```js
map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
```

The TileJSON address is `https://tiles.mapx.at/v1/tiles/{tileset}.json`. Former German names still work until 2027-05-31, see [Migration](/api/migration/).

## Next

- [Maps guide](/guides/maps/)
- [Self-host PMTiles](/guides/self-host-pmtiles/)
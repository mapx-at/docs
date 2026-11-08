# Tilesets

Alle mapx-Tilesets: Typ, Zoombereich und Datum. Ein Datenblatt pro Tileset.

Jedes Tileset ist als TileJSON und XYZ-Tiles über die [Tiles API](/de/api/tiles/) verfügbar und als eine unveränderliche [PMTiles](https://docs.protomaps.com/pmtiles/)-Datei (das Quelldatum steht im Dateinamen). Alle Daten stehen unter CC BY 4.0, siehe [Attribution](/de/reference/attribution/).

<TilesetTable />

Ein Tileset per Namen verwenden:

```js
map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
```

Die TileJSON-Adresse ist `https://tiles.mapx.at/v1/tiles/{tileset}.json`. Die früheren deutschen Namen funktionieren noch bis 2027-05-31, siehe [Migration](/de/api/migration/).

## Nächste Schritte

- [Karten-Guide](/de/guides/maps/)
- [PMTiles selbst hosten](/de/guides/self-host-pmtiles/)
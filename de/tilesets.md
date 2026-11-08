# Tilesets

Alle mapx-Tilesets: Typ, Zoombereich und Datum. Ein Datenblatt pro Tileset.

Jedes Tileset ist als TileJSON und XYZ-Tiles über die [Tiles API](/de/api/tiles/) verfügbar und als eine unveränderliche [PMTiles](https://docs.protomaps.com/pmtiles/)-Datei (das Quelldatum steht im Dateinamen). Alle Daten stehen unter CC BY 4.0, siehe [Attribution](/de/reference/attribution/).

<TilesetTable />

Nicht in der Tabelle, weil es täglich ersetzt statt versioniert wird: [Sentinel-2 aktuell](/de/tilesets/sentinel2/) (`sentinel2`, Raster, z5 bis z13).

Ein Tileset per Namen verwenden:

```js
map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
```

Alle Anfragen an `tiles.mapx.at` brauchen einen Token, siehe [Zugang und Limits](/de/start/access-and-limits/). Bei MapLibre-Quellen hängt `transformRequest` den Token an, bei Dateien `?access_token=DEIN_TOKEN`.

Die TileJSON-Adresse ist `https://tiles.mapx.at/v1/tiles/{tileset}.json`. Die früheren deutschen Namen funktionieren noch bis 2027-05-31, siehe [Migration](/de/api/migration/).

## Nächste Schritte

- [Karten-Guide](/de/guides/maps/)
- [PMTiles selbst hosten](/de/guides/self-host-pmtiles/)
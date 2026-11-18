# Tilesets: Vektor- und Rasterkacheln

Source: https://docs.mapx.at/de/tilesets/

> Alle mapx-Tilesets für Österreich mit Typ, Zoombereich und Stand: Basiskarte, Kataster, Höhenlinien, Orthophoto, Gelände- und Oberflächenmodell.

Jedes Tileset ist als TileJSON und XYZ-Tiles über die [Tiles API](https://docs.mapx.at/de/api/tiles.md) verfügbar und als eine unveränderliche [PMTiles](https://docs.protomaps.com/pmtiles/)-Datei (das Quelldatum steht im Dateinamen). Alle Daten stehen unter CC BY 4.0, siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

| Tileset | Typ | Zoom | Stand |
|---|---|---|---|
| [`streets`](https://docs.mapx.at/de/tilesets/streets.md) | Vektor | z0–16 | 2026-04-01 |
| [`cadastre`](https://docs.mapx.at/de/tilesets/cadastre.md) | Vektor | z16–16 | 2026-04-01 |
| [`contours`](https://docs.mapx.at/de/tilesets/contours.md) | Vektor | z10–16 | 20250915 |
| [`terrain-rgb`](https://docs.mapx.at/de/tilesets/terrain-rgb.md) | DEM | z5–16 | 20250915 |
| [`dsm`](https://docs.mapx.at/de/tilesets/dsm.md) | DEM | z5–16 | 20250915 |
| [`orthophoto`](https://docs.mapx.at/de/tilesets/orthophoto.md) | raster | z5–18 | 2025-10-03 |
| [`orthophoto-flight-blocks`](https://docs.mapx.at/de/tilesets/orthophoto-flight-blocks.md) | Vektor | z6–17 | 2025-10-03 |
| [`rest-areas`](https://docs.mapx.at/de/tilesets/rest-areas.md) | Vektor | z7–14 | 20261107 |
| [`sharing`](https://docs.mapx.at/de/tilesets/sharing.md) | Vektor | z8–14 | 20261107 |
| [`bike-routes`](https://docs.mapx.at/de/tilesets/bike-routes.md) | Vektor | z5–14 | 20260708 |
| [`gip-poi`](https://docs.mapx.at/de/tilesets/gip-poi.md) | Vektor | z12–15 | 20260708 |
| [`sentinel2`](https://docs.mapx.at/de/tilesets/sentinel2.md) | raster | z5–z13 | täglich |

Nicht in der Tabelle, weil es täglich ersetzt statt versioniert wird: [Sentinel-2 aktuell](https://docs.mapx.at/de/tilesets/sentinel2.md) (`sentinel2`, Raster, z5 bis z13).

Ein Tileset per Namen verwenden:

```js
map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
```

Alle Anfragen an `tiles.mapx.at` brauchen einen Token, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Bei MapLibre-Quellen hängt `transformRequest` den Token an, bei Dateien `?access_token=DEIN_TOKEN`.

Die TileJSON-Adresse ist `https://tiles.mapx.at/v1/tiles/{tileset}.json`.

## Nächste Schritte

- [Karten-Guide](https://docs.mapx.at/de/guides/maps.md)
- [PMTiles selbst hosten](https://docs.mapx.at/de/guides/self-host-pmtiles.md)

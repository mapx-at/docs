# Tilesets: Vektor- und Rasterkacheln

Source: https://docs.mapx.at/de/tilesets/

> Alle mapx-Tilesets für Österreich mit Typ, Zoombereich und Stand: Basiskarte, Höhenlinien, Orthofoto, Gelände- und Oberflächenmodell.

Jedas Tileset ist als TileJSON und XYZ-Kacheln über die [Tiles API](https://docs.mapx.at/de/api/tiles.md) verfügbar und in den [Stilen](https://docs.mapx.at/de/api/styles.md) enthalten. Die Daten stehen unter CC BY 4.0 (Ausnahme: Sentinel-2), siehe [Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md).

| Tileset | Typ | Zoom | Stand |
|---|---|---|---|
| [`streets`](https://docs.mapx.at/de/tilesets/streets.md) | Vektor | z0–16 | 01.04.2026 |
| [`contours`](https://docs.mapx.at/de/tilesets/contours.md) | Vektor | z10–16 | 15.09.2025 |
| [`terrain-rgb`](https://docs.mapx.at/de/tilesets/terrain-rgb.md) | DEM | z5–16 | 15.09.2025 |
| [`dsm`](https://docs.mapx.at/de/tilesets/dsm.md) | DEM | z5–16 | 15.09.2025 |
| [`orthophoto`](https://docs.mapx.at/de/tilesets/orthophoto.md) | raster | z5–18 | 03.10.2025 |
| [`orthophoto-flight-blocks`](https://docs.mapx.at/de/tilesets/orthophoto-flight-blocks.md) | Vektor | z6–17 | 03.10.2025 |
| [`rest-areas`](https://docs.mapx.at/de/tilesets/rest-areas.md) | Vektor | z7–14 | 20.11.2026 |
| [`sharing`](https://docs.mapx.at/de/tilesets/sharing.md) | Vektor | z8–14 | 20.11.2026 |
| [`bike-routes`](https://docs.mapx.at/de/tilesets/bike-routes.md) | Vektor | z5–14 | 08.07.2026 |
| [`gip-poi`](https://docs.mapx.at/de/tilesets/gip-poi.md) | Vektor | z12–15 | 08.07.2026 |
| [`sentinel2`](https://docs.mapx.at/de/tilesets/sentinel2.md) | raster | z5–z13 | täglich |
| [`sentinel2-date`](https://docs.mapx.at/de/tilesets/sentinel2.md) | raster | z5–z13 | täglich |
| [`sentinel2-ndvi`](https://docs.mapx.at/de/tilesets/sentinel2.md) | raster | z5–z13 | täglich |
| [`sentinel2-ndwi`](https://docs.mapx.at/de/tilesets/sentinel2.md) | raster | z5–z13 | täglich |
| [`sentinel2-ndsi`](https://docs.mapx.at/de/tilesets/sentinel2.md) | raster | z5–z13 | täglich |

Die Sentinel-2-Tilesets ([Sentinel-2 aktuell](https://docs.mapx.at/de/tilesets/sentinel2.md) und vier Auswertungen) werden täglich ersetzt statt versioniert, daher steht bei ihnen „täglich“ statt eines Stichtags.

Ein Tileset per Namen verwenden:

```js
map.addSource("contours", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" });
```

Alle Anfragen an `tiles.mapx.at` brauchen einen Token, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Bei MapLibre-Quellen hängt `transformRequest` den Token an, bei Kachel-URLs `?access_token=DEIN_TOKEN`.

Die TileJSON-Adresse ist `https://tiles.mapx.at/v1/tiles/{tileset}.json`.

## Nächste Schritte

- [Karten-Guide](https://docs.mapx.at/de/guides/maps.md)
- [Styles-API](https://docs.mapx.at/de/api/styles.md)

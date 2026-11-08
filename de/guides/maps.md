# Karten


mapx liefert Karten als fertige Styles und als Tiles. Ein Style ergibt eine vollständige Karte in einer Zeile; Tilesets kommen zum Einsatz, wenn ein eigener Style entsteht.

**Voraussetzung:** eine Seite mit MapLibre GL JS, siehe [Erste Karte](/de/start/first-map/). Ein kostenloser Token ist nötig ([Zugang und Limits](/de/start/access-and-limits/)); die Codebeispiele zeigen `DEIN_TOKEN` nur dort, wo es darauf ankommt, die Übergabe steht dort.

## Einen Style verwenden

| Style | Aussehen |
|---|---|
| `streets` | Basiskarte, Straßen, Orte, Gebäude |
| `streets-dawn`, `streets-dusk`, `streets-night` | Dieselbe Karte bei anderem Licht |
| `satellite` | Orthophoto |
| `satellite-streets` | Orthophoto mit Straßen und Beschriftung |
| `outdoors` | Höhenlinien, Schummerung, Wege |

```js
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  center: [12.694, 47.074], zoom: 12, // Großglockner
});
```

Ein Style-Wechsel zur Laufzeit ersetzt Layer und Sources: `map.setStyle(url)` aufrufen. Siehe [Basiskarte wechseln](/de/examples/switch-basemap/).

## Tilesets zum eigenen Style hinzufügen

Jedes Tileset hat eine TileJSON-Adresse `https://tiles.mapx.at/v1/tiles/{tileset}.json`. MapLibre liest daraus Zoombereich, Grenzen und Attribution.

| Tileset | Typ | Inhalt |
|---|---|---|
| `streets` | vector | Straßen, Bahn, Gewässer, Grenzen, Orte, Gebäude |
| `cadastre` | vector | Grundstücksgrenzen (z16) |
| `contours` | vector | Höhenlinien |
| `terrain-rgb`, `dsm` | raster-dem | Gelände- und Oberflächenhöhe |
| `orthophoto` | raster | Luftbild, bis z18 |

```js
map.on("load", () => {
  map.addSource("cadastre", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" });
  map.addLayer({ id: "parcels", type: "line", source: "cadastre", "source-layer": "parcel_l",
                 paint: { "line-color": "#e11d48", "line-width": 1 } });
});
```

Layernamen und Felder: [Tilesets](/de/tilesets/). Eigene Daten lassen sich genauso mit einer `geojson`-Source hinzufügen.

## Wetter- und Verkehrs-Overlays

Raster-Wetter-Tiles haben eine Zeitdimension; das TileJSON von `temperature`, `precipitation`, `wind`, `humidity`, `cloud` oder `pressure` listet die verfügbaren Zeiten. Dasselbe URL-Schema bedient die Luftqualitätsebenen `aqi` (Tagesindex), `no2`, `o3`, `pm10`, `pm25` und `dust` (Saharastaub), siehe [Air Quality API](/de/api/air-quality/). Die amtlichen Wetterwarnungen sind GeoJSON (`/v1/weather/warnings`, als `geojson`-Source einbinden und nach `level` einfärben).

```js
map.addSource("temp", { type: "raster", url: "https://api.mapx.at/v1/weather/tiles/temperature.json", tileSize: 256 });
map.addLayer({ id: "temp", type: "raster", source: "temp", paint: { "raster-opacity": 0.6 } });
```

Die Verkehrsebene ist eine Vektor-Source, siehe [Verkehr](/de/guides/traffic/). Beispiel: [Wetter-Overlay](/de/examples/weather-overlay/).

## Andere Clients

Die Style- und TileJSON-Formate funktionieren in MapLibre, in Mapbox-GL-JS-Clients mit eigenen URLs, in Leaflet mit Vektor-Plugin, in OpenLayers und in QGIS. Für Clients ohne Vektor-Unterstützung liefern [Static Tiles](/de/api/styles/) den gerenderten Style als Raster-Tiles:

```
https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}
```

## Offline und Massennutzung

Jedes Tileset ist auch eine einzelne PMTiles-Datei. Einmal herunterladen statt Tiles abzurufen: [PMTiles selbst hosten](/de/guides/self-host-pmtiles/).

> Die Datenquelle auf der Karte anzeigen. MapLibre macht das automatisch, wenn ein Style oder TileJSON geladen wird, siehe [Attribution](/de/reference/attribution/).


## Nächste Schritte

- [Basiskarte gestalten](/de/guides/style-the-basemap/)
- [Tilesets](/de/tilesets/)
- [Styles API](/de/api/styles/) und [Tiles API](/de/api/tiles/)


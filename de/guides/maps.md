# Karten-Guide: Vektorkarte und Styles

Source: https://docs.mapx.at/de/guides/maps/

> Eine mapx-Vektorkarte in MapLibre GL JS anzeigen, Styles wechseln, Tilesets und Overlays hinzufügen, auch Wetter- und Verkehrsebenen.

mapx liefert Karten als fertige Styles und als Tiles. Ein Style ergibt eine vollständige Karte in einer Zeile; Tilesets kommen zum Einsatz, wenn ein eigener Style entsteht.

**Voraussetzung:** eine Seite mit MapLibre GL JS, siehe [Erste Karte](https://docs.mapx.at/de/start/first-map.md). Ein kostenloser Token ist nötig ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Codebeispiele zeigen `DEIN_TOKEN` nur dort, wo es darauf ankommt. Das Style-Beispiel zeigt die Übergabe mit `transformRequest`, die späteren Ausschnitte setzen sie voraus.

## Einen Style verwenden

| Style | Aussehen |
|---|---|
| `streets` | Basiskarte, Straßen, Orte, Gebäude |
| `streets-dawn`, `streets-dusk`, `streets-night` | Dieselbe Karte bei anderem Licht |
| `satellite` | Orthophoto |
| `satellite-streets` | Orthophoto mit Straßen und Beschriftung |
| `outdoors` | Höhenlinien, Schummerung, Wege |

```js
const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  // Token an alle Anfragen an mapx.at hängen
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
  center: [12.694, 47.074], zoom: 12, // Großglockner
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```

Ein Style-Wechsel zur Laufzeit ersetzt Layer und Sources: `map.setStyle(url)` aufrufen. Siehe [Basiskarte wechseln](https://docs.mapx.at/de/examples/switch-basemap.md).

## Tilesets zum eigenen Style hinzufügen

Jedes Tileset hat eine TileJSON-Adresse `https://tiles.mapx.at/v1/tiles/{tileset}.json`. MapLibre liest daraus Zoombereich, Grenzen und Attribution.

| Tileset | Typ | Inhalt |
|---|---|---|
| `streets` | vector | Straßen, Bahn, Gewässer, Grenzen, Orte, Gebäude |
| `contours` | vector | Höhenlinien |
| `terrain-rgb`, `dsm` | raster-dem | Gelände- und Oberflächenhöhe |
| `orthophoto` | raster | Luftbild, bis z18 |

```js
map.on("load", () => {
  map.addSource("contours", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" });
  map.addLayer({ id: "contours", type: "line", source: "contours", "source-layer": "contour_l",
                 paint: { "line-color": "#92400e", "line-width": 0.6 } });
});
```

Layernamen und Felder: [Tilesets](https://docs.mapx.at/de/tilesets/index.md). Eigene Daten lassen sich genauso mit einer `geojson`-Source hinzufügen.

## Wetter- und Verkehrs-Overlays

Raster-Wetter-Tiles haben eine Zeitdimension; das TileJSON von `temperature`, `precipitation`, `wind`, `humidity`, `cloud` oder `pressure` listet die verfügbaren Zeiten. Dasselbe URL-Schema bedient die Luftqualitätsebenen `aqi` (Tagesindex), `no2`, `o3`, `pm10`, `pm25` und `dust` (Saharastaub), siehe [Air Quality API](https://docs.mapx.at/de/api/air-quality.md).

```js
map.addSource("temp", { type: "raster", url: "https://api.mapx.at/v1/weather/tiles/temperature.json", tileSize: 256 });
map.addLayer({ id: "temp", type: "raster", source: "temp", paint: { "raster-opacity": 0.6 } });
```

Die Verkehrsebene ist eine Vektor-Source, siehe [Verkehr](https://docs.mapx.at/de/guides/traffic.md). Beispiel: [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md).

## Andere Clients

Die Style- und TileJSON-Formate funktionieren in MapLibre, in Mapbox-GL-JS-Clients mit eigenen URLs und in [QGIS](https://docs.mapx.at/de/guides/qgis.md). Für Clients ohne Vektor-Unterstützung liefern [Static Tiles](https://docs.mapx.at/de/api/styles.md) den gerenderten Style als Raster-Tiles; so binden [OpenLayers](https://docs.mapx.at/de/guides/openlayers.md) und [Leaflet](https://docs.mapx.at/de/guides/leaflet.md) die Basiskarte ein:

```
https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=DEIN_TOKEN
```

## Offline und Massennutzung

Kacheln und Tilesets gibt es nur über die API: Style, TileJSON oder `{z}/{x}/{y}`. Ganze Dateien zum Herunterladen bietet mapx nicht an. Für viele Anfragen die Kacheln im Client cachen, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

> Die Datenquelle auf der Karte anzeigen. MapLibre macht das automatisch, wenn ein Style oder TileJSON geladen wird; der Code oben klappt sie hinter den Knopf „i“ ein, siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

## Nächste Schritte

- [Basiskarte gestalten](https://docs.mapx.at/de/guides/style-the-basemap.md)
- [Tilesets](https://docs.mapx.at/de/tilesets/index.md)
- [Styles API](https://docs.mapx.at/de/api/styles.md) und [Tiles API](https://docs.mapx.at/de/api/tiles.md)

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
| `light`, `dark` | Ruhige, fast einfarbige Grundkarte (hell bzw. dunkel) für eigene Daten |
| `outdoors` | Höhenlinien, Schummerung, Wege |
| `outdoors-winter` | Wie `outdoors` in Schneefarben, Skigebiete und Seilbahnen/Lifte hervorgehoben |
| `relief` | Reine Reliefkarte: Höhenstufen von grünen Tälern über Gelb, Ocker und Braun bis Grau und Weiß in der Gipfelregion, kräftige Schummerung, Seen und Flüsse, wenige Namen (Orte, Gipfel mit Höhe, Gewässer); keine Straßen und Gebäude |
| `relief-contours` | Wie `relief`, zusätzlich Höhenlinien mit Höhenangaben |
| `relief-light`, `relief-dark` | Nur graue Schummerung auf hellem bzw. dunklem Grund, sehr zurückhaltend, als Hintergrund für eigene Daten |

Die Relief-Styles brauchen MapLibre GL JS ab 5.6 (Layertyp `color-relief`) und gibt es nur für die Karte im Browser, nicht als Static Images oder Static Tiles. Sie zeigen nur Österreich (Geländemodell BEV ALS-DTM 1 m, z5 bis z16), außerhalb bleibt die Fläche einfarbig.

### 3D-Styles

| Style | Inhalt |
|---|---|
| `outdoors-3d`, `outdoors-winter-3d`, `relief-3d` | `outdoors`, `outdoors-winter` bzw. `relief` über dem echten Geländemodell, mit Himmel und Horizont |
| `satellite-3d`, `satellite-streets-3d` | Luftbild über dem Gelände, pur bzw. mit Straßen und Namen |
| `streets-3d`, `streets-night-3d` | Stadtkarte mit Gebäuden als Körper (Höhen aus dem BEV-Gebäudemodell) und Szenenlicht, bei Tag bzw. bei Nacht |
| `light-3d`, `dark-3d` | Ruhige Grundkarte hell bzw. dunkel mit grauen bzw. anthrazitfarbenen Gebäudekörpern, für eigene Daten |

Die 3D-Styles schalten den Himmel (`sky`) und das Gelände (`terrain` auf dem Geländemodell `terrain-rgb`, in den Stadt-Styles ohne Überhöhung) im Style ein und bringen eine geneigte Startansicht mit (`pitch` 72, Stadt 68, dazu `bearing`). MapLibre übernimmt Neigung und Richtung aus dem Style nur, wenn beim Erzeugen der Karte keine Kameraposition gesetzt ist; mit `center` und `zoom` daher `pitch` und `bearing` mitgeben. MapLibre erlaubt standardmäßig höchstens 60 Grad Neigung, dabei liegt der Horizont meist außerhalb des Bildes. Wer den Himmel sehen will, setzt `maxPitch` und `pitch` selbst:

```js
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets-3d",
  center: [16.3731, 48.2085], zoom: 15,
  maxPitch: 80, pitch: 68, bearing: -20, // Himmel im Bild
});
```

Gebäudekörper erscheinen ab etwa Zoom 13 (zusammengefasste Gebäude mit mittlerer Höhe, ab Zoom 15 jedes Gebäude mit eigener Höhe). Die Höhen stammen aus dem BEV-Gebäudemodell, fehlende aus den Höhenmodellen (Oberfläche minus Gelände); hohe Türme stehen als eigene Teile (`building_part_a`) auf den Gebäuden. Gelände gibt es nur für Österreich. Die 3D-Styles gibt es nur für die Karte im Browser (MapLibre GL JS), nicht als Static Images oder Static Tiles; dort den gleichnamigen Style ohne `-3d` verwenden.

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

> Die Datenquelle auf der Karte anzeigen. MapLibre macht das automatisch, wenn ein Style oder TileJSON geladen wird, mit `mapx.at` (Link) als erstem Eintrag; bitte nicht entfernen; der Code oben klappt sie hinter den Knopf „i“ ein, siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

## Nächste Schritte

- [Basiskarte gestalten](https://docs.mapx.at/de/guides/style-the-basemap.md)
- [Tilesets](https://docs.mapx.at/de/tilesets/index.md)
- [Styles API](https://docs.mapx.at/de/api/styles.md) und [Tiles API](https://docs.mapx.at/de/api/tiles.md)

# Karten-Guide: Vektorkarte und Stile

Source: https://docs.mapx.at/de/guides/maps/

> Eine mapx-Vektorkarte in MapLibre GL JS anzeigen, Stile wechseln, Tilesets und Overlays hinzufügen, auch Wetter- und Verkehrsebenen.

mapx liefert Karten als fertige Stile und als Kacheln. Ein Stil ergibt eine vollständige Karte in einer Zeile; Tilesets kommen zum Einsatz, wenn ein eigener Stil entsteht.

**Voraussetzung:** eine Seite mit MapLibre GL JS, siehe [Erste Karte](https://docs.mapx.at/de/start/first-map.md). Ein kostenloser Token ist nötig ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Codebeispiele zeigen `DEIN_TOKEN` nur dort, wo es darauf ankommt. Das Stil-Beispiel zeigt die Übergabe mit `transformRequest`, die späteren Ausschnitte setzen sie voraus.

**PMTiles-Protokoll:** Die mapx-Stile laden ihre Kacheln aus PMTiles-Dateien. Die Seite muss deshalb `pmtiles.js` einbinden und einmal `maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile)` aufrufen, sonst bleibt die Karte leer (Konsole: `URL scheme "pmtiles" is not supported`). Alle Ausschnitte unten setzen das voraus. Eigene Stile, die nur TileJSON-Quellen (`https://tiles.mapx.at/v1/tiles/{tileset}.json`) verwenden, brauchen es nicht.

## Einen Stil verwenden

| Stil | Aussehen |
|---|---|
| `streets` | Basiskarte, Straßen, Orte, Gebäude |
| `streets-dawn`, `streets-dusk`, `streets-night` | Dieselbe Karte bei anderem Licht |
| `satellite` | Orthofoto |
| `satellite-current` | Aktuelles Satellitenbild (Sentinel-2) statt des Orthofotos |
| `satellite-streets` | Orthofoto mit Straßen und Beschriftung |
| `light`, `dark` | Ruhige, fast einfarbige Grundkarte (hell bzw. dunkel) für eigene Daten |
| `outdoors` | Höhenlinien, Schummerung, Wege |
| `outdoors-winter` | Wie `outdoors` in Schneefarben, Skigebiete und Seilbahnen/Lifte hervorgehoben |
| `relief` | Reine Reliefkarte: Höhenstufen von grünen Tälern über Gelb, Ocker und Braun bis Grau und Weiß in der Gipfelregion, kräftige Schummerung, Seen und Flüsse, wenige Namen (Orte, Gipfel mit Höhe, Gewässer); keine Straßen und Gebäude |
| `relief-contours` | Wie `relief`, zusätzlich Höhenlinien mit Höhenangaben |
| `relief-light`, `relief-dark` | Nur graue Schummerung auf hellem bzw. dunklem Grund, sehr zurückhaltend, als Hintergrund für eigene Daten |
| `traffic-day`, `traffic-night` | Karte mit den aktuellen Verkehrsmeldungen und Sperren (Tag bzw. Nacht), siehe [Verkehr](https://docs.mapx.at/de/guides/traffic.md) |

Die Reliefstile brauchen MapLibre GL JS ab 5.6 (Ebenentyp `color-relief`) und gibt es nur für die Karte im Browser. Sie zeigen nur Österreich (Geländemodell BEV ALS-DTM 1 m, z5 bis z16), außerhalb bleibt die Fläche einfarbig.

**Als Bild:** [Statische Kartenbilder](https://docs.mapx.at/de/guides/static-images.md) und [Static Tiles](https://docs.mapx.at/de/api/styles.md) rendern `streets` (mit den Tageszeit-Varianten), `light`, `dark`, `satellite`, `satellite-streets`, `outdoors` (die volle Vektorkarte) und `outdoors-winter`. Nicht als Bild verfügbar sind `traffic-day` und `traffic-night` (Antwort `404` mit Hinweis auf `streets` bzw. `streets-night`), `satellite-current`, die Reliefstile und alle `-3d`-Stile.

### 3D-Stile

| Stil | Inhalt |
|---|---|
| `outdoors-3d`, `outdoors-winter-3d`, `relief-3d` | `outdoors`, `outdoors-winter` bzw. `relief` über dem echten Geländemodell, mit Himmel und Horizont |
| `satellite-3d`, `satellite-streets-3d` | Luftbild über dem Gelände, pur bzw. mit Straßen und Namen |
| `streets-3d`, `streets-night-3d` | Stadtkarte mit Gebäuden als Körper (Höhen aus dem BEV-Gebäudemodell) und Szenenlicht, bei Tag bzw. bei Nacht |
| `light-3d`, `dark-3d` | Ruhige Grundkarte hell bzw. dunkel mit grauen bzw. anthrazitfarbenen Gebäudekörpern, für eigene Daten |

Die 3D-Stile schalten den Himmel (`sky`) und das Gelände (`terrain` auf dem Geländemodell `terrain-rgb`, in den Stadt-Stilen ohne Überhöhung) im Stil ein und bringen eine geneigte Startansicht mit (`pitch` 72, Stadt 68, dazu `bearing`). MapLibre übernimmt Neigung und Richtung aus dem Stil nur, wenn beim Erzeugen der Karte keine Kameraposition gesetzt ist; mit `center` und `zoom` daher `pitch` und `bearing` mitgeben. MapLibre erlaubt standardmäßig höchstens 60 Grad Neigung, dabei liegt der Horizont meist außerhalb des Bildes. Wer den Himmel sehen will, setzt `maxPitch` und `pitch` selbst:

```js
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile); // einmal je Seite, siehe oben
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets-3d",
  // transformRequest wie im Beispiel weiter unten
  center: [16.3731, 48.2085], zoom: 15,
  maxPitch: 78, pitch: 74, bearing: -20, // ab etwa 72 Grad Neigung kommt der Himmel ins Bild
});
// Stadt-Stile: Gebäude bis zum Horizont (nach jedem Laden eines Stils neu setzen)
map.on("style.load", () => map.setSourceTileLodParams(3, 6));
```

Bei starker Neigung lädt MapLibre für die Ferne Kacheln kleinerer Zoomstufen. Unterhalb von Zoom 12 enthalten sie keine Gebäude, die Gebäudekörper hören dann im Hintergrund auf. `setSourceTileLodParams(3, 6)` begrenzt die Zahl der Zoomstufen im Bild auf drei, sodass die Körper bis zum Horizont reichen; dafür lädt die Karte etwas mehr Kacheln. Für die Gelände- und Luftbild-Stile ist das nicht nötig.

In den Stadt-Stilen `maxPitch` nicht über 78 setzen: Bei stärkerer Neigung schneidet MapLibre das Gelände am Horizont ab, die Gebäude auf Hügeln dahinter zeichnet es aber weiter, sie scheinen dann zu schweben.

Gebäudekörper wachsen zwischen Zoom 12 und 13 aus dem Boden und stehen dann auch in der Ferne einer geneigten Karte (zusammengefasste Gebäude mit mittlerer Höhe, ab Zoom 15 jedes Gebäude mit eigener Höhe). Die Höhen stammen aus dem BEV-Gebäudemodell, fehlende aus den Höhenmodellen (Oberfläche minus Gelände); hohe Türme stehen als eigene Teile (`building_part_a`) auf den Gebäuden. Gelände gibt es nur für Österreich. Die 3D-Stile gibt es nur für die Karte im Browser (MapLibre GL JS), nicht als Statische Kartenbilder oder Static Tiles; dort den Stil ohne `-3d` verwenden (für `relief-3d` gibt es auch kein Bild).

```js
const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile); // die Stile laden PMTiles
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: "https://api.mapx.at/v1/styles/outdoors",
  // Token an alle Anfragen an mapx.at hängen
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
  center: [12.694, 47.074], zoom: 12, // Großglockner
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```

Ein Stilwechsel zur Laufzeit ersetzt Ebenen und Quellen: `map.setStyle(url)` aufrufen. Eigene Quellen und Ebenen danach neu hinzufügen (zum Beispiel im Ereignis `style.load`). Siehe [Basiskarte wechseln](https://docs.mapx.at/de/examples/switch-basemap.md).

## Tilesets zum eigenen Stil hinzufügen

Jedas Tileset hat eine TileJSON-Adresse `https://tiles.mapx.at/v1/tiles/{tileset}.json`. MapLibre liest daraus Zoombereich, Grenzen und Quellenangabe.

| Tileset | Typ | Inhalt |
|---|---|---|
| `streets` | vector | Straßen, Bahn, Gewässer, Grenzen, Orte, Gebäude |
| `contours` | vector | Höhenlinien |
| `terrain-rgb`, `dsm` | raster-dem | Gelände- und Oberflächenhöhe |
| `orthophoto` | raster | Luftbild, bis z18 |

```js
map.on("load", () => {
  map.addSource("mx-contours", { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" });
  map.addLayer({ id: "mx-contours", type: "line", source: "mx-contours", "source-layer": "contour_l",
                 paint: { "line-color": "#92400e", "line-width": 0.6 } });
});
```

Die mapx-Stile belegen die Quellennamen `streets`, `contours`, `terrain-rgb`, `dsm`, `orthophoto`, `traffic` und weitere. Wer eine Quelle zu einem mapx-Stil hinzufügt, wählt einen eigenen Namen (hier `mx-contours`); `outdoors` zeigt Höhenlinien ohnehin schon. Ebenennamen und Felder: [Tilesets](https://docs.mapx.at/de/tilesets/index.md). Eigene Daten lassen sich genauso mit einer `geojson`-Quelle hinzufügen, ein vollständiges Beispiel zeigt [GeoJSON-Overlay](https://docs.mapx.at/de/examples/geojson-overlay.md).

## Wetter- und Verkehrs-Overlays

Raster-Wetter-Kacheln haben eine Zeitdimension; das TileJSON von `temperature`, `precipitation`, `wind`, `humidity`, `cloud` oder `pressure` listet die verfügbaren Zeiten im Feld `mapx:time.times`. Die Kachel-URL enthält standardmäßig `/now/`; für eine andere Zeit setzt du einen Eintrag aus dieser Liste anstelle von `now` ein (Vorlage in `mapx:time.template`). Dasselbe URL-Schema bedient die Luftqualitätsebenen `aqi` (Tagesindex), `no2`, `o3`, `pm10`, `pm25` und `dust` (Saharastaub), siehe [Air Quality API](https://docs.mapx.at/de/api/air-quality.md).

```js
map.addSource("temp", { type: "raster", url: "https://api.mapx.at/v1/weather/tiles/temperature.json", tileSize: 256 });
map.addLayer({ id: "temp", type: "raster", source: "temp", paint: { "raster-opacity": 0.6 } });
```

Die Verkehrsebene ist eine Vektor-Source, siehe [Verkehr](https://docs.mapx.at/de/guides/traffic.md). Beispiel: [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md).

## Andere Clients

Das TileJSON-Format funktioniert in MapLibre und in [QGIS](https://docs.mapx.at/de/guides/qgis.md). Die Stile selbst laden PMTiles-Dateien (`pmtiles://`) und laufen daher nur in Clients, die dieses Protokoll kennen, etwa MapLibre mit `pmtiles.js`. Für Clients ohne Vektor-Unterstützung liefern [Static Tiles](https://docs.mapx.at/de/api/styles.md) den gerenderten Stil als Rasterkacheln; so binden [OpenLayers](https://docs.mapx.at/de/guides/openlayers.md) und [Leaflet](https://docs.mapx.at/de/guides/leaflet.md) die Basiskarte ein:

```
https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=DEIN_TOKEN
```

## Offline und Massennutzung

Kacheln und Tilesets gibt es nur über die API: Stil, TileJSON oder `{z}/{x}/{y}`. Ganze Dateien zum Herunterladen bietet mapx nicht an. Für viele Anfragen die Kacheln im Client cachen, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

> Die Datenquelle auf der Karte anzeigen. MapLibre macht das automatisch, wenn ein Stil oder TileJSON geladen wird, mit `mapx.at` (Link) als erstem Eintrag; bitte nicht entfernen; der Code oben klappt sie hinter den Knopf „i“ ein, siehe [Quellenangabe](https://docs.mapx.at/de/reference/attribution.md).

## Nächste Schritte

- [Basiskarte gestalten](https://docs.mapx.at/de/guides/style-the-basemap.md)
- [Tilesets](https://docs.mapx.at/de/tilesets/index.md)
- [Styles API](https://docs.mapx.at/de/api/styles.md) und [Tiles API](https://docs.mapx.at/de/api/tiles.md)

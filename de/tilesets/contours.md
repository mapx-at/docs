# Höhenlinien-Kachelsatz für Österreich

Source: https://docs.mapx.at/de/tilesets/contours/

> Höhenlinien für Österreich als Vektorkacheln, z10 bis z16, 100 m bis 10 m Abstand, abgeleitet aus dem BEV ALS-DTM 1 m.

Höhenlinien für ganz Österreich als Vektorkacheln, abgeleitet aus dem [DTM](https://docs.mapx.at/de/tilesets/terrain-rgb.md). Es gibt vier Generalisierungsstufen mit eigenem Linienabstand, eigener Glättung und eigenen Indexlinien.

**Nutzungen**

- **Wander- und Outdoorkarten**: Höhenlinien mit Beschriftung der Indexlinien (https://docs.mapx.at/de/guides/overlay-contours/)
- **Überlagerung**: Linien über Basiskarte oder Orthofoto (https://docs.mapx.at/de/guides/overlay-contours/)
- **Routenplanung**: Steigung und Geländeform lesen
- **Planungskarten**: Höhenschichten in Fachkarten
- **QGIS**: Vektorkacheln im Desktop-GIS (https://docs.mapx.at/de/guides/qgis/)

## Kennwerte

**Kachelsatz `contours`**

| Kennwert | Wert |
|---|---|
| Stichtag | `15.09.2025` |
| Format | `mvt` |
| Zoomstufen | `z10–z16` |
| Gebiet (WGS84) | `9,5311 / 46,3723 / 17,1605 / 49,0210 (W/S/O/N)` |
| Lizenz | `CC BY 4.0` |

## Live-Karte

## Einbinden

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/contours.json" } },
    layers: [{ id: "contour_l", type: "line", source: "v", "source-layer": "contour_l", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/contours/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/contours.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/contours/14/8860/5730.pbf
```

## Ebenen-Referenz

| `min_zoom` | Zoom | Abstand | Indexlinie | Raster | Glättung (Gauß-σ) | Mindestlänge |
|---|---|---|---|---|---|---|
| 10 | 10–11 | 100 m | 500 m | 20 m | 60 m | 600 m |
| 12 | 12–13 | 50 m | 200 m | 10 m | 25 m | 250 m |
| 14 | 14–15 | 20 m | 100 m | 4 m | 10 m | 100 m |
| 16 | ab 16 | 10 m | 50 m | 2 m | 4 m | 40 m |

Unter z10 gibt es keine Linien. Jede Kachel enthält genau eine Stufe (`min_zoom`), ein Filter nach `min_zoom` ist nicht nötig.

```js
{ id: "hl", type: "line", source: "v", "source-layer": "contour_l",
  paint: { "line-color": "#b5651d", "line-width": ["case", ["==", ["get", "index"], 1], 1.4, 0.6] } }
{ id: "hl-text", type: "symbol", source: "v", "source-layer": "contour_l",
  filter: ["==", ["get", "index"], 1],
  layout: { "symbol-placement": "line", "text-field": ["concat", ["get", "ele"], " m"], "text-size": 10 } }
```

### contour_l

Geometrie: **Linie**, Zoom: **z10–16**, Ebene: `contour_l`. Höhenlinien (Isolinien) in vier Detailstufen.

| Feld | Typ | Beschreibung |
|---|---|---|
| `ele` | `number` | Höhe der Linie in Metern über Adria. |
| `index` | `number` | 1 bei Hauptlinien (Zählkurven), sonst 0. Werte: `0` = Normale Höhenlinie; `1` = Hauptlinie (Zählkurve, meist beschriftet) |
| `min_zoom` | `number` | Detailstufe der Höhenlinie; entspricht der Kartenzoomstufe, ab der sie gezeigt wird (Äquidistanz 100 m, 50 m, 20 m, 10 m). Werte: `10` = Zoom 10–11, Äquidistanz 100 m (Hauptlinie alle 500 m); `12` = Zoom 12–13, Äquidistanz 50 m (Hauptlinie alle 200 m); `14` = Zoom 14–15, Äquidistanz 20 m (Hauptlinie alle 100 m); `16` = Zoom 16, Äquidistanz 10 m (Hauptlinie alle 50 m) |

## Quelle und Aktualität

- Quelle: BEV ALS-DTM 1 m, Stichtag 15.09.2025 (wie das DTM).

## Hinweise

- Durch die Glättung liegt die Linie in groben Stufen bewusst nicht exakt auf der Rohhöhe.
- Linien enden am Rand der Daten (Staatsgrenze).
- In Steilwänden liegen 10-m-Linien sehr dicht; Ausdünnen ist Aufgabe des Stils.
- Beschriftung nur über `ele`, es gibt keine eigenen Label-Punkte.

## Quellenangabe

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus dem TileJSON:

```html
mapx.at; Höhenlinien: mapx.at, Datenquelle <a href="https://data.bev.gv.at/geonetwork/srv/ger/catalog.search#/metadata/5ce253fc-b7c5-4362-97af-6556c18a45d9">BEV – ALS-DTM 1 m</a> (Stichtag 15.09.2025), CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

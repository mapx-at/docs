# Höhenlinien

Höhenlinien für Österreich als Vektorkacheln, z10 bis z16, 100 m bis 10 m Abstand, abgeleitet aus dem BEV ALS-DTM 1 m.

export const t = data.tilesets.hoehenlinien;
export const url = `https://tiles.mapx.at/${t.datei}`;

Höhenlinien aus dem [DTM](/daten/dtm/), in vier Generalisierungsstufen. Jede Stufe hat eigenen Linienabstand, eigene Glättung und eigene Indexlinien.

## Kennwerte

<Kennwerte id="hoehenlinien" />

## Live-Karte

<ProduktKarte id="hoehenlinien" center={[12.694, 47.074]} zoom={13} />

## Layer

<LayerTabelle id="hoehenlinien" info={{ contour_l: "<code>ele</code> Höhe in m, <code>index</code> 1 = Indexlinie, <code>lvl</code> Stufe (10, 12, 14, 16)" }} />

## Stufen

| `lvl` | Zoom | Abstand | Indexlinie | Raster | Glättung (Gauß σ) | Mindestlänge |
|---|---|---|---|---|---|---|
| 10 | 10–11 | 100 m | 500 m | 20 m | 60 m | 600 m |
| 12 | 12–13 | 50 m | 200 m | 10 m | 25 m | 250 m |
| 14 | 14–15 | 20 m | 100 m | 4 m | 10 m | 100 m |
| 16 | ab 16 | 10 m | 50 m | 2 m | 4 m | 40 m |

Die Staffelung folgt dem früheren basemap.at-Höhenlinien-Cache. Unter z10 gibt es keine Linien. Im Style nach `lvl` filtern, sonst liegen in z12 und darüber mehrere Stufen übereinander,
falls der Client überzoomt.

<Code lang="js" code={`{ id: "hl", type: "line", source: "v", "source-layer": "contour_l",
  paint: { "line-color": "#b5651d", "line-width": ["case", ["==", ["get", "index"], 1], 1.4, 0.6] } }
{ id: "hl-text", type: "symbol", source: "v", "source-layer": "contour_l",
  filter: ["==", ["get", "index"], 1],
  layout: { "symbol-placement": "line", "text-field": ["concat", ["get", "ele"], " m"], "text-size": 10 } }`} />

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/hoehenlinien.json" } },
    layers: [{ id: "contour_l", type: "line", source: "v", "source-layer": "contour_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/hoehenlinien/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/hoehenlinien.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/hoehenlinien/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Datenherkunft

- Quelle: BEV ALS-DTM 1 m, Stichtag 15.09.2025 (wie das DTM).
- Erzeugt mit `crates/mapx-contours` (Rust, ohne GDAL): Quelle je Stufe auf das Zielraster mitteln, Gauß-Glättung, Marching Squares, Douglas-Peucker je Kachel.
- Stehgewässer ab 1 ha (GGN) sind maskiert, die Linien enden am Ufer.

## Genauigkeit und Grenzen

- Geprüft: Lauf aus URL und aus lokalen Blöcken byte-gleich, Fortsetzen nach Abbruch byte-gleich zum Lauf am Stück. Gegenüber `gdal_contour` weichen an Sätteln 3 von 362 Linien ab (anderer Sattelentscheid).
- Absolute Lagegenauigkeit gegen das DTM: **nicht gemessen**. Durch die Glättung liegt die Linie in groben Stufen bewusst nicht exakt auf der Rohhöhe.
- Linien enden am Gebiets- bzw. DTM-Rand (Staatsgrenze), Lücke höchstens ein Rasterpixel der Stufe.
- In Steilwänden (z. B. Dachstein-Südwand) liegen 10-m-Linien sehr dicht; Ausdünnen ist Aufgabe des Styles.
- Beschriftung nur über `ele`, es gibt keine eigenen Label-Punkte.

## Versionen

Datei mit Stichtag und Hash im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei | SHA-256 |
|---|---|---|
| {t.stand} | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.hoehenlinien.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
# Höhenlinien

Höhenlinien für Österreich als Vektorkacheln, z10 bis z16, 100 m bis 10 m Abstand, abgeleitet aus dem BEV ALS-DTM 1 m.

export const t = data.tilesets.hoehenlinien;
export const url = `https://tiles.mapx.at/${t.datei}`;

Höhenlinien für ganz Österreich als Vektorkacheln, abgeleitet aus dem [DTM](/daten/dtm/). Es gibt vier Generalisierungsstufen mit eigenem Linienabstand, eigener Glättung und eigenen Indexlinien.

<Nutzungen items={[{t:"Wander- und Outdoorkarten",d:"Höhenlinien mit Beschriftung der Indexlinien",href:"/anleitungen/hoehenlinien-ueberlagern/"},{t:"Überlagerung",d:"Linien über Basiskarte oder Orthophoto",href:"/anleitungen/hoehenlinien-ueberlagern/"},{t:"Routenplanung",d:"Steigung und Geländeform lesen"},{t:"Planungskarten",d:"Höhenschichten in Fachkarten"},{t:"QGIS",d:"Vektorkacheln im Desktop-GIS",href:"/anleitungen/qgis/"}]} />

## Kennwerte

<Kennwerte id="hoehenlinien" />

## Live-Karte

<ProduktKarte id="hoehenlinien" center={[12.694, 47.074]} zoom={13} />

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/contours.json" } },
    layers: [{ id: "contour_l", type: "line", source: "v", "source-layer": "contour_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/contours/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/contours.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/contours/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Layer-Referenz

| `lvl` | Zoom | Abstand | Indexlinie |
|---|---|---|---|
| 10 | 10–11 | 100 m | 500 m |
| 12 | 12–13 | 50 m | 200 m |
| 14 | 14–15 | 20 m | 100 m |
| 16 | ab 16 | 10 m | 50 m |

Jede Stufe hat eigene Glättung und Mindestlänge der Linien. Unter z10 gibt es keine Linien. Im Style nach `lvl` filtern, sonst liegen in z12 und darüber mehrere Stufen übereinander,
falls der Client überzoomt.

<Code lang="js" code={`{ id: "hl", type: "line", source: "v", "source-layer": "contour_l",
  paint: { "line-color": "#b5651d", "line-width": ["case", ["==", ["get", "index"], 1], 1.4, 0.6] } }
{ id: "hl-text", type: "symbol", source: "v", "source-layer": "contour_l",
  filter: ["==", ["get", "index"], 1],
  layout: { "symbol-placement": "line", "text-field": ["concat", ["get", "ele"], " m"], "text-size": 10 } }`} />

### contour_l

<FeldReferenz ts="hoehenlinien" layer="contour_l" />

## Quelle und Aktualität

- Quelle: BEV ALS-DTM 1 m, Stichtag 15.09.2025 (wie das DTM).
- Die Linien sind je Stufe geglättet und generalisiert. Stehgewässer ab 1 ha sind maskiert, die Linien enden am Ufer.

## Hinweise

- Durch die Glättung liegt die Linie in groben Stufen bewusst nicht exakt auf der Rohhöhe.
- Linien enden am Rand der Daten (Staatsgrenze).
- In Steilwänden liegen 10-m-Linien sehr dicht; Ausdünnen ist Aufgabe des Styles.
- Beschriftung nur über `ele`, es gibt keine eigenen Label-Punkte.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.hoehenlinien.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
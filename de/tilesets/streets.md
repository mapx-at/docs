# Basiskarte

Vektor-Basiskarte Österreich aus offenen Behördendaten: Nutzung, Gebäude, Straßen, Bahn, Gewässer, Grenzen, Namen, Hausnummern. z0 bis z16.

export const t = data.tilesets.basis;
export const url = `https://tiles.mapx.at/${t.datei}`;

Die Basiskarte vereint die offenen Daten von BEV, GIP, Umweltbundesamt und BMLUK in 18 Layern. Sie enthält keine OpenStreetMap-Daten. Kataster, Höhenlinien und Relief sind eigene Produkte und lassen sich darüberlegen.

<Nutzungen items={[{t:"Eigener Kartenstil",d:"Die Layer nach Wunsch stylen",href:"/de/guides/style-the-basemap/"},{t:"Web-Karten und Apps",d:"Vektorkarte mit eigener Gestaltung bis z16"},{t:"Planungs- und Navigationshintergrund",d:"Straßen, Bahn, Gewässer, Grenzen"},{t:"Hausnummern und Namen",d:"Adressen, Orte, Berge und Gewässernamen"},{t:"Überlagerung",d:"Kataster, Höhenlinien und Relief darüber",href:"/de/guides/overlay-contours/"}]} />

## Kennwerte

<Kennwerte id="basis" />

## Live-Karte

<ProduktKarte id="basis" center={[15.334, 48.227]} zoom={14} />

Die Farben der Live-Karte sind automatisch vergeben (Fläche, Linie, Punkt je Layer), kein fertiger Kartenstil. Klick zeigt die Attribute.

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
    layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/de/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/tiles/streets.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/tiles/streets/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Layer-Referenz

Layername endet auf `_a` (Fläche), `_l` (Linie) oder `_p` (Punkt). `min_zoom` am Objekt sagt, ab welcher Stufe es gedacht ist; `sort_rank` ordnet innerhalb des Layers.

<LayerTabelle id="basis" info={{'landuse_a': 'Bodennutzung aus der DKM: residential, garden, farmland, vineyard, alpine_meadow, forest, glacier, rock, water, wetland, transport, quarry, cemetery, recreation', 'building_a': 'Gebäude (DKM) mit Höhe aus dem DLM; <code>kind</code> building oder church, school, hospital, station, castle, hut …; z12–14 zusammengefasst', 'road_l': 'Straßen und Wege (GIP): highway, major_road, minor_road, service, track, path; <code>structure</code> bridge/tunnel', 'rail_l': 'Bahn (GIP): rail, narrow_gauge, rack, subway, tram, funicular, ferry', 'aerialway_l': 'Seilbahnen und Lifte (DLM): cable_car, gondola, chair_lift, drag_lift …', 'transit_p': 'Bahnhöfe, Halte, Liftstationen', 'airport_p': 'Flugplätze (BEV DLM Verkehr): airport, airfield, heliport, military', 'ski_a': 'Skigebiete (Umweltbundesamt, Stand 2017)', 'boundary_l': 'Grenzen (BEV VGD): country, state, district, municipality; <code>left</code>/<code>right</code> Gebiet links/rechts', 'boundary_label_p': 'Namen von Bundesland, Bezirk, Gemeinde; <code>code</code> Kennzahl', 'place_p': 'Orte (BEV Geographische Namen): city, town, village, hamlet, suburb, locality; <code>capital</code>', 'natural_p': 'Berge, Sättel, Täler, Flurnamen, Burgen, Ruinen …', 'poi_p': 'Kirchen, Gipfelkreuze und Einrichtungen (BEV DLM, in Wien zusätzlich Wien-OGD): Schulen, Spitäler, Hütten, Tankstellen, Museen …', 'housenumber_p': 'Hausnummern (BEV Adressregister): <code>kind</code> building = Gebäudekoordinate, parcel = Grundstücksadresse', 'water_l': 'Fließgewässer (GGN): river, stream, canal, ditch', 'water_a': 'Stehgewässer (GGN): lake, reservoir, pond', 'water_label_l': 'Gewässernamen entlang der Linie', 'water_label_p': 'Seenamen, ein Punkt je See'}} />

Alle Felder und Werte jedes Layers: [Basiskarte: Layer-Referenz](/de/reference/style-spec/).

## Quelle und Aktualität

| Thema | Quelle | Stand |
|---|---|---|
| Nutzung, Gebäude | BEV Digitale Katastralmappe (DKM), Höhen aus dem DLM | 01.04.2026 |
| Straßen, Bahn | GIP-OGD (Graphenintegrationsplattform) | 30.04.2026 |
| Lifte, Stationen, POI | BEV DLM, Wien-OGD (nur Wien) | |
| Grenzen | BEV Verwaltungsgrenzen (VGD) | 01.10.2026 |
| Orte, Namen, Gipfel | BEV Geographische Namen | 23.03.2026 |
| Hausnummern | BEV Adressregister | 01.10.2026 |
| Gewässer | BMLUK Gesamtgewässernetz (GGN v19) | |
| Skigebiete | Umweltbundesamt | 2017 |
| Flugplätze | BEV DLM Verkehr (CC BY 4.0) | |


## Hinweise

- Nicht enthalten: Halte von Privatbahnen sowie U-Bahn- und Straßenbahnhaltestellen; kleine Flugfelder fehlen teils; Gewässer kennen kein „verrohrt“. Skigebiete haben den Stand 2017.
- Ausland: praktisch leer (nur Bodensee und Gewässer bis knapp hinter die Grenze).
- Glyphen und Schriften für Beschriftungen liefert mapx nicht; der Style braucht eigene.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.basis.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/de/reference/attribution/).
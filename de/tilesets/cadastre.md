# Kataster

Grundstücksgrenzen aus der Digitalen Katastralmappe des BEV als Vektorkacheln, z16, mit Katastralgemeinde und Grundstücksnummer.

export const t = data.tilesets.kataster;
export const url = `https://tiles.mapx.at/${t.datei}`;

Die Grenzen aller Grundstücke als Linien, mit Katastralgemeinde-Nummer und Grundstücksnummer, aus der Digitalen Katastralmappe des BEV. Das Tileset hat nur z16; Clients zoomen darüber auf.

<Nutzungen items={[{t:"Grundstückssuche",d:"Grenzen und Nummern auf der Karte"},{t:"Planung und Vorprüfung",d:"Grundstücksgrenzen über Orthophoto legen"},{t:"Immobilien- und Standortkarten",d:"Parzellen als Kartenebene"},{t:"Überlagerung",d:"Kataster über Basiskarte oder Luftbild",href:"/de/guides/style-the-basemap/"},{t:"QGIS",d:"Vektorkacheln im Desktop-GIS",href:"/de/guides/qgis/"}]} />

## Kennwerte

<Kennwerte id="kataster" />

## Live-Karte

<ProduktKarte id="kataster" center={[15.334, 48.227]} zoom={16.5} />

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" } },
    layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/de/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/cadastre/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/cadastre.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/cadastre/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

Weil es nur z16 gibt, im Style `minzoom: 16` setzen; die Quelle mit `maxzoom: 16` überzoomt automatisch.

## Layer-Referenz

Die Linien sind Grenzen, keine Flächen: Ein Klick trifft die Grenzlinie und liefert das Grundstück, zu dem sie gehört. Eigentümer und Fläche sind nicht enthalten.

### parcel_l

<FeldReferenz ts="kataster" layer="parcel_l" />

## Quelle und Aktualität

- Quelle: BEV **Digitale Katastralmappe (DKM)**, Stichtag 01.04.2026.

## Hinweise

> Die Darstellung ist nicht rechtsverbindlich. Für Grenzfragen gilt der Kataster beim Vermessungsamt.

- Die Abbildung der Grenzen ist nicht rechtsverbindlich, maßgeblich ist der Kataster beim Vermessungsamt.
- Unter z16 gibt es keine Kacheln.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.kataster.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/de/reference/attribution/).
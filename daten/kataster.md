# Kataster

Grundstücksgrenzen aus der Digitalen Katastralmappe des BEV als Vektorkacheln, z16, mit Katastralgemeinde und Grundstücksnummer.

export const t = data.tilesets.kataster;
export const url = `https://tiles.mapx.at/${t.datei}`;

Die Grenzen aller Grundstücke als Linien, mit Katastralgemeinde-Nummer und Grundstücksnummer. Das Tileset hat nur z16; Clients zoomen darüber auf.

<Aside type="caution">Die Darstellung ist nicht rechtsverbindlich. Für Grenzfragen gilt der Kataster beim Vermessungsamt.</Aside>

## Kennwerte

<Kennwerte id="kataster" />

## Live-Karte

<ProduktKarte id="kataster" center={[15.334, 48.227]} zoom={16.5} />

## Layer

<LayerTabelle id="kataster" info={{ parcel_l: "<code>kgnr</code> Katastralgemeinde-Nummer, <code>gstnr</code> Grundstücksnummer (Bauflächen mit Punkt, z. B. <code>.211</code>)" }} />

Die Linien sind Grenzen, keine Flächen: Ein Klick trifft die Grenzlinie und liefert das Grundstück, zu dem sie gehört. Eigentümer und Fläche sind nicht enthalten.

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/mapx-kataster.json" } },
    layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/mapx-kataster/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/mapx-kataster.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/mapx-kataster/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

Weil es nur z16 gibt, im Style `minzoom: 16` setzen; die Quelle mit `maxzoom: 16` überzoomt automatisch.

## Datenherkunft

- Quelle: BEV **Digitale Katastralmappe (DKM)**, Stichtag 01.04.2026.
- Die Lage ist über das amtliche Transformationsgitter nach ETRS89 umgerechnet.
- Die Grenzen sind aus den Grundstücksflächen abgeleitet.

## Hinweise

- Die Abbildung der Grenzen ist nicht rechtsverbindlich, maßgeblich ist der Kataster beim Vermessungsamt.
- Unter z16 gibt es keine Kacheln.

## Versionen

Datei mit Stichtag im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.kataster.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
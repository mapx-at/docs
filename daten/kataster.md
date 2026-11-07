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
- Umrechnung aus den Gauß-Krüger-Streifen M28/M31/M34 nach ETRS89 mit dem amtlichen NTv2-Gitter (Probepunkte 0,000 m Abweichung).
- Grenzen aus den Grundstücksflächen abgeleitet (`ST_Boundary`).

## Genauigkeit und Grenzen

- Kanten gegen Orthofoto (Pilot Melk): 0,2 m (Stift Melk) bis 2,4 m (freies Feld mit schwachem Kontrast); das misst die Abbildung, nicht die Rechtsgrenze.
- Prüfungen sind für die Pilotbezirke belegt, Vollständigkeit für ganz Österreich ist **nicht dokumentiert**.
- Unter z16 gibt es keine Kacheln.

## Versionen

Datei mit Stichtag und Hash im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei | SHA-256 |
|---|---|---|
| {t.stand} | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.kataster.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
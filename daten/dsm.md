# DSM Oberflächenmodell

Digitales Oberflächenmodell Österreich mit Gebäuden und Bewuchs als Terrain-RGB (0,1 m), z5 bis z16, aus dem BEV ALS-DOM 1 m.

export const t = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das DSM ist das Gegenstück zum [DTM](/daten/dtm/): Es zeigt die **Oberfläche** mit Gebäuden, Bäumen und Brücken. Kodierung, Kachelgröße und Zoomstufen sind identisch,
alle Clients und Formeln des DTM gelten unverändert.

## Kennwerte

<Kennwerte id="dsm" />

## Live-Karte

<DtmMap id="dsm" />

Der Klick dekodiert das Pixel der Kachel (Formel unter [Terrain-RGB](/konzepte/terrain-rgb/)).

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/dsm.json", tileSize: 512, encoding: "mapbox" } },
    terrain: { source: "dsm", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/start/)).
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/dsm/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="QGIS">
    XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/dsm/{z}/{x}/{y}.webp`. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/dsm.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/dsm/14/8860/5730.webp`} />
  </TabItem>
</Tabs>

Höhe per HTTP: [Höhe und Profil](/api/hoehe/) mit `model=dsm`.

## Datenherkunft

- Quelle: BEV **ALS-DOM 1 m** (Airborne Laserscanning, Oberfläche), Geonetwork-Serie `cced2282-cfc5-42eb-9acb-b1c799522fcd`, Stichtag **15.09.2025**, 55 Quellkacheln.
- Gleiche Rust-Pipeline wie das DTM (`crates/mapx-rgb`): Reprojektion EPSG:3035 → 3857 bilinear, Mapbox-Kodierung 0,1 m, 512 px, WebP verlustfrei.
- Übersichtsstufen z15 bis z5 als 2×2-Mittel im Höhenraum.

## Genauigkeit und Grenzen

- Quantisierung 0,1 m, maximal z16 (≈ 0,8 m pro Pixel bei 48° N), darüber zoomt der Client auf.
- Gemessen (4 Mio. Pixel, Innsbruck): DSM minus DTM im Median 2,9 m, P95 21 m, Maximum 56 m; nur 0,07 % der Pixel liegen mehr als 0,5 m unter dem DTM.
- Eine eigene Höhenprüfung gegen das Quell-DSM wie beim DTM ist **nicht dokumentiert**.
- Aufnahmezeitraum der Befliegungen: nicht dokumentiert. Gebäude und Bewuchs entsprechen dem Flugzeitpunkt, nicht dem Stichtag.
- Außerhalb der Daten ist der Wert eine glatte Fortsetzung, keine Messung (wie beim DTM).
- **nDSM** (Gebäude- und Bewuchshöhe = DSM − DTM) gibt es nicht als eigenes Tileset; es lässt sich im Client aus beiden Kachelsätzen rechnen.

## Versionen

Datei mit Stichtag und Hash im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei | SHA-256 |
|---|---|---|
| {t.stand} | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dsm.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
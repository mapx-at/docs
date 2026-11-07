# DSM Oberflächenmodell

Digitales Oberflächenmodell Österreich mit Gebäuden und Bewuchs als Terrain-RGB (0,1 m), z5 bis z16, aus dem BEV ALS-DOM 1 m.

export const t = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das DSM ist das Gegenstück zum [DTM](/daten/dtm/): Es zeigt die Oberfläche mit Gebäuden, Bäumen und Brücken. Kodierung, Kachelgröße und Zoomstufen sind identisch, alle Clients und Formeln des DTM gelten unverändert.

<Nutzungen items={[{t:"Gebäude- und Bewuchshöhe",d:"nDSM aus DSM minus DTM im Client"},{t:"Solarplanung",d:"Dach- und Verschattungshöhen als Grundlage"},{t:"Sichtbarkeit und Verschattung",d:"Oberfläche inklusive Bebauung und Wald"},{t:"3D-Stadt- und Landschaftsansicht",d:"Oberfläche als Terrain im Browser",href:"/anleitungen/gelaende-3d/"},{t:"Höhenprofile",d:"Profil über der Oberfläche",href:"/anleitungen/hoehenprofil/"}]} />

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

## Quelle und Aktualität

Das DSM entsteht aus dem ALS-DOM 1 m des BEV (Airborne Laserscanning, Oberfläche). Der Stichtag steht in den Kennwerten und im Dateinamen; eine Datei ändert sich nie, ein neuer Stand ist eine neue Datei.

- Quelle: BEV ALS-DOM 1 m, Stichtag 15.09.2025.
- Gleiche Kodierung wie das DTM: Mapbox-Terrain-RGB mit 0,1 m Quantisierung, 512 px, WebP verlustfrei.
- Die Übersichtsstufen z15 bis z5 sind im Höhenraum gemittelt.

## Hinweise

- Quantisierung 0,1 m, maximal z16 (≈ 0,8 m pro Pixel bei 48° N), darüber zoomt der Client auf.
- Die Genauigkeit entspricht der amtlichen Quelle des BEV.
- Gebäude und Bewuchs entsprechen dem Zeitpunkt der Befliegung, nicht dem Stichtag.
- Außerhalb der Daten ist der Wert eine glatte Fortsetzung, keine Messung (wie beim DTM).
- Gebäude- und Bewuchshöhe (nDSM) ergeben sich im Client aus DSM minus DTM.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dsm.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
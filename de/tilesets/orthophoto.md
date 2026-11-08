# Orthophoto

Luftbild Österreich des BEV als WebP-Kacheln, z5 bis z18, 20 cm, mit eindeutiger Flugblock-Auswahl je Pixel.

export const t = data.tilesets.dop;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das Orthophoto des BEV, neu gekachelt in Web-Mercator bis z18 (20 cm, die native Auflösung der Quelle). Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](/de/tilesets/orthophoto-flight-blocks/).

<Nutzungen items={[{t:"Luftbild-Hintergrund",d:"Satellitenkarten-Ersatz für Web und Apps"},{t:"3D-Gelände",d:"Orthophoto auf dem DTM",href:"/de/guides/3d-terrain/"},{t:"Planung und Dokumentation",d:"Zustand vor Ort in 20 cm"},{t:"Grundstücke prüfen",d:"Kataster über dem Luftbild"},{t:"QGIS",d:"Luftbild als XYZ- oder PMTiles-Layer",href:"/de/guides/qgis/"}]} />

## Kennwerte

<Kennwerte id="dop" />

## Live-Karte

<ProduktKarte id="dop" overlay="dop_flugblock" center={[15.334, 48.227]} zoom={14} />

Klick zeigt den Flugblock am Punkt (Operat, Flugzeitraum).

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { ortho: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 } },
    layers: [{ id: "ortho", type: "raster", source: "ortho" }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/de/start/)).
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="QGIS">
    XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp`. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/tiles/orthophoto.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/tiles/orthophoto/14/8860/5730.webp`} />
  </TabItem>
</Tabs>

## Quelle und Aktualität

- Quelle: BEV **Digitales Orthophoto (DOP) RGB**, 20 cm, Aufnahmen 2022 bis 2025.
- **Ein Flug je Pixel:** Wo sich Befliegungen überlappen, gewinnt die mit dem späteren Ende des Flugzeitraums. Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](/de/tilesets/orthophoto-flight-blocks/).

## Hinweise

- Ganz Österreich ist bedeckt.
- Zwischen Bildflügen sind Farbnähte sichtbar, es gibt keine Farbangleichung.
- An den Grenzen zu DE, CH, LI, IT, SI, CZ und SK reicht das Bild teils mehr als 3 km ins Ausland; an der Grenze zu Ungarn endet es an der Grenze.
- Unter z18 mischen sich an Blockgrenzen Pixel beider Flüge; eindeutig ist die Zuordnung nur in z18.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dop.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/de/reference/attribution/).
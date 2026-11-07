# Orthophoto

Luftbild Österreich des BEV als WebP-Kacheln, z5 bis z18, 20 cm, mit eindeutiger Flugblock-Auswahl je Pixel.

export const t = data.tilesets.dop;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das Orthophoto des BEV, neu gekachelt in Web-Mercator bis z18 (20 cm, die native Auflösung der Quelle). Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](/daten/flugbloecke/).

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
    sources: { ortho: { type: "raster", url: "https://tiles.mapx.at/v1/dop-ortho.json", tileSize: 512 } },
    layers: [{ id: "ortho", type: "raster", source: "ortho" }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://${url}"` (Protokoll wie im [Schnellstart](/start/)).
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="QGIS">
    XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/dop-ortho/{z}/{x}/{y}.webp`. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/dop-ortho.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/dop-ortho/14/8860/5730.webp`} />
  </TabItem>
</Tabs>

## Datenherkunft

- Quelle: BEV **Digitales Orthophoto (DOP) RGB**, 20 cm, Aufnahmen 2022 bis 2025.
- z18 ist über das amtliche Transformationsgitter (ETRS89 ↔ MGI) entzerrt; die Stufen z17 bis z5 sind daraus abgeleitet.
- **Ein Flug je Pixel:** Wo sich Befliegungen überlappen, gewinnt die mit dem späteren Ende des Flugzeitraums. Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](/daten/flugbloecke/).

## Hinweise

- Ganz Österreich ist bedeckt.
- Zwischen Bildflügen sind Farbnähte sichtbar, es gibt keine Farbangleichung.
- An den Grenzen zu DE, CH, LI, IT, SI, CZ und SK reicht das Bild teils mehr als 3 km ins Ausland; an der Grenze zu Ungarn endet es an der Grenze.
- Unter z18 mischen sich an Blockgrenzen Pixel beider Flüge; eindeutig ist die Zuordnung nur in z18.

## Versionen

Datei mit Stichtag im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei |
|---|---|
| {t.stand} | <code>{t.datei}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dop.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
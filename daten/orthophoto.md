# Orthophoto

Luftbild Österreich des BEV als WebP-Kacheln, z5 bis z18, 20 cm, mit eindeutiger Flugblock-Auswahl je Pixel.

export const t = data.tilesets.dop;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das Orthophoto des BEV, neu gekachelt in Web-Mercator bis z18 (20 cm, die native Auflösung der Quelle). Welcher Flug ein Pixel liefert, beschreibt das Tileset [Flugblöcke](/daten/flugbloecke/).

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

- Quelle: BEV **Digitales Orthophoto (DOP) RGB**, 32 Operat-Mosaike, 20 cm, Aufnahmen 2022 bis 2025, gelesen direkt aus den Cloud-Optimized GeoTIFFs des BEV.
- z18 je Kachel exakt über das amtliche Gitter (ETRS89 ↔ MGI) entzerrt, Neuabtastung Catmull-Rom, Maske gewichtet.
- z17 bis z5 als 2×2-Mittel, WebP Qualität 80 ab z15, darunter 75. Leere Kacheln entfallen.
- **Ein Besitzer je Pixel:** Wo sich Operate überlappen, gewinnt das mit dem späteren Ende des Flugzeitraums (dann früherer Beginn, dann Archivnummer). Keine Überblendung, harte Naht an der Operatgrenze.

## Genauigkeit und Grenzen

- Geprüft: Pixel gegen Flugblock-Polygon 0 Abweichungen (35 400 Pixel, dazu 830 000 Randpixel). Georeferenz gegen basemap.at bestätigt, kein Versatz erkennbar; Farbabweichung (MAE RGB) 4,5 bis 6,6, bei gleichem Flug 3 bis 5.
- Ganz Österreich ist bedeckt (0 km² ohne Bild).
- Zwischen Operaten bleiben Farbnähte sichtbar, es gibt keine Farbangleichung.
- An den Grenzen zu DE, CH, LI, IT, SI, CZ und SK reicht das Bild teils mehr als 3 km ins Ausland (insgesamt rund 3640 km²); an der Grenze zu Ungarn endet es an der Grenze.
- Unter z18 mischen sich an Blockgrenzen Pixel beider Flüge; eindeutig ist die Zuordnung nur in z18.
- Der Stand in der Datei (`{t.stand}`) und der Lieferstand der Quell-Mosaike (15.04.2026, Feld `stichtag` der Flugblöcke) weichen voneinander ab.

## Versionen

Datei mit Stichtag und Hash im Namen, sie ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stand | Datei | SHA-256 |
|---|---|---|
| {t.stand} | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Alle Änderungen: [Changelog](/changelog/).

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

<Code lang="html" code={data.tilesets.dop.metadata.attribution || "BEV, CC BY 4.0"} />

Mehr: [Lizenz](/lizenz/).
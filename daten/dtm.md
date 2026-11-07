# DTM Geländemodell

Digitales Geländemodell Österreich als Terrain-RGB-Kacheln (Mapbox-Kodierung, 0,1 m), z5 bis z16, aus dem BEV ALS-DTM 1 m.

export const t = data.tilesets.relief;
export const dsm = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das DTM ist ein Höhenkachelsatz für ganz Österreich: Geländehöhe ohne Gebäude und Bewuchs, kodiert als Farbe (Terrain-RGB) und
damit direkt für `raster-dem`, Schummerung und 3D-Gelände in MapLibre, Mapbox GL und kompatiblen Clients verwendbar.

## 1. Kennwerte

<Kennwerte id="relief" />

Die Tabelle wird beim Build aus `viewer/tiles-manifest.json` und dem PMTiles-Header der Datei erzeugt (Stand der Erzeugung: {data.erzeugt}).

## 2. Live-Karte

<DtmMap id="relief" />

Der Klick liest die Kachel der aktuellen Zoomstufe (maximal z16) per Range-Request, dekodiert das Pixel am Klickpunkt mit der
Formel aus Abschnitt 4 und zeigt Höhe und RGB-Wert. Bei eingeschaltetem 3D steht zusätzlich der Wert von `queryTerrainElevation` dabei.

## 3. Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`import * as maplibregl from "maplibre-gl";

maplibregl.addProtocol("pmtiles", new Protocol().tile);
const map = new maplibregl.Map({
  container: "map",
  center: [12.694, 47.074], zoom: 11, pitch: 60,
  style: {
    version: 8,
    sources: { dem: { type: "raster-dem", url: "pmtiles://${url}", tileSize: 512, encoding: "mapbox", maxzoom: 16 } },
    terrain: { source: "dem", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dem" }]
  }
});
map.on("click", (e) => console.log(map.queryTerrainElevation(e.lngLat)));`} />
  </TabItem>
  <TabItem label="Leaflet">
    Leaflet kennt kein `raster-dem`. Über den XYZ-Endpunkt ([API](/api/kacheln/#xyz)) kommen die Rohkacheln; sie zeigen die kodierte Farbe, nicht
    eine Karte. Für Höhenwerte die Pixel selbst dekodieren (Abschnitt 4).
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, minZoom: 5, maxNativeZoom: 16, maxZoom: 20,
  crossOrigin: true,
  attribution: 'BEV – ALS-DTM 1 m, CC BY 4.0'
}).addTo(map);`} />
  </TabItem>
  <TabItem label="OpenLayers">
    <Code lang="js" code={`import TileLayer from "ol/layer/WebGLTile.js";

// Bänder sind in OpenLayers auf 0..1 normiert, daher * 255
const hoehe = ["+", -10000, ["*", 0.1, ["+",
  ["*", 65536 * 255, ["band", 1]], ["*", 256 * 255, ["band", 2]], ["*", 255, ["band", 3]]]]];

new TileLayer({
  source: new XYZ({
    url: "https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp",
    tileSize: 512, minZoom: 5, maxZoom: 16, crossOrigin: "anonymous",
    attributions: "BEV – ALS-DTM 1 m, CC BY 4.0"
  }),
  style: { color: ["interpolate", ["linear"], hoehe, 200, [60, 120, 60], 1500, [200, 190, 120], 3000, [255, 255, 255]] }
});`} />
  </TabItem>
  <TabItem label="QGIS">
    1. Browser, Rechtsklick auf **XYZ Tiles**, **Neue Verbindung**.
    2. URL `https://tiles.mapx.at/v1/dtm/{z}/{x}/{y}.webp`, Min. Zoom 5, Max. Zoom 16, Kachelauflösung 512.
    3. Der Layer zeigt die Rohkacheln (kodierte Farbe). Höhen ergeben sich aus der Formel in Abschnitt 4 (`(R*65536 + G*256 + B)*0,1 - 10000`), etwa im Raster-Rechner auf den drei Bändern.

    Die Anleitung ist nicht in QGIS nachvollzogen; sie folgt dem Ablauf für XYZ-Kacheln.
  </TabItem>
  <TabItem label="Python">
    <Code lang="python" code={`# pip install pmtiles pillow numpy requests
from PIL import Image
from pmtiles.reader import Reader

URL = "${url}"

def get_bytes(offset, length):
    r = requests.get(URL, headers={"Range": f"bytes={offset}-{offset + length - 1}"}, timeout=30)
    r.raise_for_status()
    return r.content

reader = Reader(get_bytes)
tile = reader.get(12, 2192, 1439)          # z, x, y (Großglockner)
rgb = np.asarray(Image.open(io.BytesIO(tile)).convert("RGB")).astype(np.uint32)
hoehe = -10000 + (rgb[..., 0] * 65536 + rgb[..., 1] * 256 + rgb[..., 2]) * 0.1
print(hoehe.shape, hoehe.min(), hoehe.max())`} />
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`# PMTiles-Header (127 Bytes) per Range-Request
curl -s -r 0-126 ${url} | xxd | head -4

# eine Kachel über den XYZ-Endpunkt
curl -s -o kachel.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp

# ganze Datei (${(t.bytes / 1e9).toFixed(1)} GB), fortsetzbar
curl -C - -O ${url}`} />
  </TabItem>
  <TabItem label="JS / Rust">
    <Code lang="js" title="JavaScript" code={`const hoehe = (r, g, b) => -10000 + (r * 65536 + g * 256 + b) * 0.1;`} />
    <Code lang="rust" title="Rust" code={`fn hoehe(r: u8, g: u8, b: u8) -> f64 {
    -10000.0 + (u32::from(r) * 65536 + u32::from(g) * 256 + u32::from(b)) as f64 * 0.1
}
// Gegenrichtung (wie die Pipeline in crates/mapx-rgb): auf 0,1 m runden, auf 0..=16777215 begrenzen
fn rgb(h: f64) -> [u8; 3] {
    let q = ((h + 10000.0) * 10.0).round_ties_even().clamp(0.0, 16777215.0) as u32;
    [(q >> 16) as u8, (q >> 8) as u8, q as u8]
}`} />
  </TabItem>
</Tabs>

## 4. Kodierung

```
höhe_m = -10000 + (R · 65536 + G · 256 + B) · 0,1
```

- Wertebereich theoretisch −10000 m bis +1 667 721,5 m in Schritten von 0,1 m; in Österreich liegen die Werte zwischen rund 115 m (Neusiedler See) und rund 3 798 m (Großglockner).
- Gerundet wird beim Schreiben auf 0,1 m (Rundung zur geraden Zahl bei genau .5, siehe Rust-Code oben), Werte außerhalb des Bereichs werden begrenzt.
- WebP **verlustfrei**: verlustbehaftete Kompression würde die kodierten Höhen direkt verfälschen. Keine Alpha-Information nutzen, RGB-Bänder unverändert lesen (kein Farbmanagement, kein Premultiplied Alpha).
- 512 × 512 Pixel, **ohne Rand** (kein 1-px-Skirt). In MapLibre `tileSize: 512` und `encoding: "mapbox"` angeben.

Ausführlich: [Terrain-RGB](/konzepte/terrain-rgb/).

## 5. Auflösung je Zoom

Bei 47,5° N, 512-px-Kacheln, Web-Mercator:

<Aufloesung />

Die Daten stammen aus einem 1-m-Raster. z16 (≈ 0,8 m pro Pixel) ist die erste Stufe, die die Quelle nicht vergröbert; z15 würde etwa 60 % der Pixel verwerfen.
Über z16 gibt es keine Kacheln: MapLibre zoomt ab z17 die z16-Kachel auf (`maxzoom: 16` setzen), es entsteht keine zusätzliche Information.
Die Stufen z15 bis z5 sind 2×2-Mittel im Höhenraum (Dekodieren, mitteln, neu kodieren), kein Bild-Resize.

## 6. Datenherkunft

- Quelle: BEV (Bundesamt für Eich- und Vermessungswesen), **ALS-DTM 1 m**, Geonetwork-Serie `5ce253fc-b7c5-4362-97af-6556c18a45d9`, Airborne Laserscanning, nur Gelände.
- 55 Quellkacheln à 50 × 50 km als COG in EPSG:3035 (ETRS89-LAEA), Stichtag **15.09.2025**.
- Reprojektion nach EPSG:3857 **bilinear** (nicht Lanczos: Überschwinger an steilen Klippen würden die Höhen selbst verfälschen).
- Erzeugt mit der Rust-Pipeline `crates/mapx-rgb` (ohne GDAL).

## 7. Genauigkeit und Grenzen

**Belegt** (Quelle: `relief/README.md`, `relief/test/hoehen_check.py`, Vault `mapx/Terrain-RGB-Rust-Test`):

- Quantisierung 0,1 m.
- z16 gegen das bilinear interpolierte Quell-DTM an drei Prüfpunkten: Jauerling 960,10 m (Δ −0,03 m), Donau bei Melk 203,90 m (Δ −0,04 m), Stift Melk 248,30 m (Δ +0,02 m).
- Rust-Render gegen die Quelle auf zehn Kacheln (Alpen und Melk): höchstens 0,05 m Abweichung, das ist der Rundungsfehler.
- Kantenzusammenführung: Pixel am Rand der Quellkacheln werden aus den gültigen Nachbarn renormiert.

**Grenzen:**

- DTM zeigt das Gelände ohne Gebäude, Bäume und Brücken. Für die Oberfläche gibt es das [DSM](#8-varianten-und-verwandte).
- **Kein Messwert außerhalb der Daten.** Das Gebiet des Kachelsatzes ist das Rechteck 9,53–17,17° O, 46,37–49,02° N. Pixel ohne Quelldaten (außerhalb Österreichs, Lücken) sind eine glatte Fortsetzung des Geländes, keine Messung. Im Test (Durchstich Melk gegen das DTM außerhalb des Gebiets) lag der Fehler dieser Fortsetzung im Mittel bei 11 m, am Übergang springt die Höhe im Median um bis zu 17 m (steiles Gelände, Maximum 46 m).
- Abweichung der Übersichtsstufen z15 bis z5 gegenüber der Quelle: **nicht gemessen** (nur Mittelbildung im Höhenraum, verglichen wurde Python gegen Rust, nicht gegen die Quelle).
- Gesamtgenauigkeit über ganz Österreich (RMSE gegen unabhängige Referenzpunkte): **nicht gemessen**. Die Lagegenauigkeit und vertikale Genauigkeit des BEV-Modells selbst gibt der Datenanbieter an, hier nicht wiederholt.
- Der Anteil von z15 und z16 an der Dateigröße ist nicht gemessen.

## 8. Varianten und Verwandte

| Produkt | Inhalt | Datei | Größe |
|---|---|---|---|
| DTM (diese Seite) | Gelände | <code>{t.datei}</code> | {(t.bytes / 1e9).toFixed(1)} GB |
| [DSM](/daten/dsm/) | Oberfläche (inkl. Gebäude, Bewuchs), gleiche Kodierung, BEV ALS-DOM 1 m | <code>{dsm.datei}</code> | {(dsm.bytes / 1e9).toFixed(1)} GB |
| [Höhenlinien](/daten/hoehenlinien/) | Vektor, z10–z16 | siehe Produktseite | |

Ein nDSM (DSM minus DTM, Bewuchs- und Gebäudehöhe) ist angedacht, aber nicht verfügbar.

## 9. Versionen und Changelog

Jede Datei trägt Stichtag und Hash im Namen und ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stichtag | Datei | SHA-256 |
|---|---|---|
| 15.09.2025 | <code>{t.datei}</code> | <code>{t.sha256}</code> |

Prüfen: `curl -s {url} | sha256sum` (lädt die ganze Datei). Alle Änderungen: [Changelog](/changelog/).

## 10. Attribution

Pflicht nach CC BY 4.0. Text:

```
Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0
```

HTML (wie in den PMTiles-Metadaten, MapLibre übernimmt es automatisch aus dem Tileset):

<Code lang="html" code={data.tilesets.relief.metadata.attribution} />

Mehr: [Lizenz](/lizenz/).

## 11. FAQ

**Warum WebP verlustfrei?** Weil jede Veränderung der Farbe eine andere Höhe bedeutet. Verlustfrei ist die einzige Wahl, die die 0,1-m-Kodierung erhält.

**Warum kein Nodata?** Ein Nodata-Wert (zum Beispiel Höhe 0) würde in Hillshade und 3D als Loch oder Graben erscheinen. Fehlende Pixel werden deshalb vor dem Kodieren aus dem Füllraster (flächentreues Mittel, Push-Pull-Fortsetzung) glatt ergänzt. Kacheln ganz ohne Quelldaten gibt es nicht (Lücke im PMTiles). Wo ein Wert vorliegt, ist er daher nicht automatisch ein Messwert, siehe Abschnitt 7.

**Unterschied zu Mapbox und MapTiler?** Gleiche Kodierung wie `mapbox-terrain-dem-v1` (Mapbox-Formel, 512 px, aber ohne 1-px-Rand). Mapbox liefert Daten bis z14 (PNG, nur über SDKs, Abrechnung je Request), MapTiler nach unserem Kenntnisstand bis etwa z12 (nicht nachgeprüft). mapx geht bis z16, ist nur auf Österreich begrenzt, verlustfrei, schlüsselfrei und als ganze Datei herunterladbar.

**Warum sehe ich Nähte in der Schummerung?** Kacheln mit 1-px-Rand (514 px) erzeugen mit `tileSize: 512` Nähte. Diese Kacheln sind randlos 512 × 512, `tileSize: 512` ist richtig.
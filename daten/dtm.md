# DTM Geländemodell

Digitales Geländemodell Österreich als Terrain-RGB-Kacheln (Mapbox-Kodierung, 0,1 m), z5 bis z16, aus dem BEV ALS-DTM 1 m.

export const t = data.tilesets.relief;
export const dsm = data.tilesets.dsm;
export const url = `https://tiles.mapx.at/${t.datei}`;

Das DTM ist ein Höhenkachelsatz für ganz Österreich: Geländehöhe ohne Gebäude und Bewuchs, kodiert als Farbe (Terrain-RGB) und
damit direkt für `raster-dem`, Schummerung und 3D-Gelände in MapLibre, Mapbox GL und kompatiblen Clients verwendbar.

## 1. Kennwerte

<Kennwerte id="relief" />

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
    <Code lang="bash" code={`# eine Kachel über den XYZ-Endpunkt
curl -s -o kachel.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp

# ganze Datei, fortsetzbar
curl -C - -O ${url}`} />
  </TabItem>
  <TabItem label="JS / Rust">
    <Code lang="js" title="JavaScript" code={`const hoehe = (r, g, b) => -10000 + (r * 65536 + g * 256 + b) * 0.1;`} />
    <Code lang="rust" title="Rust" code={`fn hoehe(r: u8, g: u8, b: u8) -> f64 {
    -10000.0 + (u32::from(r) * 65536 + u32::from(g) * 256 + u32::from(b)) as f64 * 0.1
}
// Gegenrichtung: auf 0,1 m runden, auf 0..=16777215 begrenzen
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
- Gerundet wird beim Schreiben auf 0,1 m (Rundung zur geraden Zahl bei genau .5, siehe Code oben), Werte außerhalb des Bereichs werden begrenzt.
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

- Quelle: BEV (Bundesamt für Eich- und Vermessungswesen), **ALS-DTM 1 m**, Airborne Laserscanning, nur Gelände, Stichtag **15.09.2025**.
- Die Quelle liegt in EPSG:3035 (ETRS89-LAEA) und ist bilinear nach EPSG:3857 umgerechnet; das vermeidet Überschwinger an steilen Klippen.

## 7. Genauigkeit und Hinweise

- Quantisierung 0,1 m. Die Abweichung zur Quelle liegt im Rundungsbereich (höchstens 0,05 m).
- Die Lage- und Höhengenauigkeit entspricht der amtlichen Quelle des BEV.
- Das DTM zeigt das Gelände ohne Gebäude, Bäume und Brücken. Für die Oberfläche gibt es das [DSM](#8-varianten-und-verwandte).
- **Kein Messwert außerhalb der Daten.** Das Gebiet des Kachelsatzes ist das Rechteck 9,53–17,17° O, 46,37–49,02° N. Pixel ohne Quelldaten (außerhalb Österreichs, Lücken) sind eine glatte Fortsetzung des Geländes, keine Messung.
- Die Übersichtsstufen z15 bis z5 sind im Höhenraum gemittelt.

## 8. Varianten und Verwandte

| Produkt | Inhalt | Datei |
|---|---|---|
| DTM (diese Seite) | Gelände | <code>{t.datei}</code> |
| [DSM](/daten/dsm/) | Oberfläche (inkl. Gebäude, Bewuchs), gleiche Kodierung, BEV ALS-DOM 1 m | <code>{dsm.datei}</code> |
| [Höhenlinien](/daten/hoehenlinien/) | Vektor, z10–z16 | siehe Produktseite |

Gebäude- und Bewuchshöhe (nDSM) ergeben sich im Client aus DSM minus DTM.

## 9. Versionen und Changelog

Jede Datei trägt den Stichtag im Namen und ändert sich nie; ein neuer Stand ist eine neue Datei.

| Stichtag | Datei |
|---|---|
| 15.09.2025 | <code>{t.datei}</code> |

Alle Änderungen: [Changelog](/changelog/).

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

**Warum kein Nodata?** Ein Nodata-Wert (zum Beispiel Höhe 0) würde in Hillshade und 3D als Loch oder Graben erscheinen. Fehlende Pixel werden deshalb vor dem Kodieren glatt aus der Umgebung ergänzt. Kacheln ganz ohne Quelldaten gibt es nicht (Lücke im PMTiles). Wo ein Wert vorliegt, ist er daher nicht automatisch ein Messwert, siehe Abschnitt 7.

**Unterschied zu Mapbox?** Gleiche Kodierung wie `mapbox-terrain-dem-v1` (Mapbox-Formel, 512 px, aber ohne 1-px-Rand). mapx geht bis z16, ist offen und schlüsselfrei, verlustfrei und als ganze Datei herunterladbar; die Abdeckung ist auf Österreich begrenzt.

**Warum sehe ich Nähte in der Schummerung?** Kacheln mit 1-px-Rand (514 px) erzeugen mit `tileSize: 512` Nähte. Diese Kacheln sind randlos 512 × 512, `tileSize: 512` ist richtig.
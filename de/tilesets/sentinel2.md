# Sentinel-2 aktuell

Immer aktuelles, wolkenfreies Satellitenbild Österreichs aus Copernicus Sentinel-2 als WebP-Kacheln, z5 bis z13, 10 m, täglich um 02:00 aktualisiert.

Ein Satellitenbild Österreichs, das sich täglich fortschreibt: Für jedes Pixel gilt ein **robuster Wert aus den jüngsten wolken- und schattenfreien Sentinel-2-Aufnahmen** (Rolling Best-Pixel-Composite). Auflösung 10 m, Web-Mercator, WebP, 512 px, z5 bis z13 (darüber überzoomt der Client). Das Aufnahmedatum jedes Pixels wird im Bestand mitgeführt.

<Nutzungen items={[{t:"Aktueller Hintergrund",d:"Satellitenansicht, die nicht Jahre alt ist"},{t:"Veränderungen sehen",d:"Schneelage, Ernte, Windwurf, Baustellen im 10-m-Raster"},{t:"Übersicht und Planung",d:"Landschaft, Vegetation und Gewässer im Überblick"}]} />

## Einbinden

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  center: [15.334, 48.227], zoom: 11,
  style: {
    version: 8,
    sources: { s2: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/sentinel2.json", tileSize: 512 } },
    layers: [{ id: "s2", type: "raster", source: "s2" }]
  }
});`} />
    TileJSON liefert Zoombereich, Grenzen und die Attribution. Der fertige Stil `satellit-aktuell` (Sentinel-2 mit Straßen und Namen) steht unter `/v1/styles/satellit-aktuell`.
  </TabItem>
  <TabItem label="Leaflet">
    <Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/tiles/sentinel2/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, minZoom: 5, maxNativeZoom: 13, maxZoom: 18, crossOrigin: true,
  attribution: "Contains modified Copernicus Sentinel data 2026"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/tiles/sentinel2.json
curl -s -o kachel.webp https://tiles.mapx.at/v1/tiles/sentinel2/13/4444/2839.webp`} />
  </TabItem>
</Tabs>

> Das Tileset wird täglich ersetzt (festes Tileset `sentinel2`, kein versionierter Dateiname). Die TileJSON nennt den Stand im Feld `mapx:stand` (jüngste eingepflegte Szene); PMTiles direkt: `https://tiles.mapx.at/sentinel2.pmtiles` (Cache 5 Minuten).

## TileJSON, Zoom und Aktualisierung

| | |
|---|---|
| TileJSON | `https://tiles.mapx.at/v1/tiles/sentinel2.json` |
| Kacheln | `https://tiles.mapx.at/v1/tiles/sentinel2/{z}/{x}/{y}.webp` |
| Zoom | z5 bis z13 (6,4 m je Pixel bei z13 auf 47,5 Grad Nord, Quelle 10 m) |
| Format | WebP, 512 px, Web-Mercator |
| Abdeckung | Österreich (Kacheln am Rand reichen ins Ausland) |
| Aktualisierung | täglich um 02:00; neue Szenen werden in den Bestand eingepflegt, danach wird die Datei atomar ersetzt |

## Quelle und Aufbau

- Quelle: **Sentinel-2 L2A** (Bodenreflexion) der ESA/Copernicus als Cloud-Optimized-GeoTIFF über Element84 Earth Search (STAC, AWS Open Data). Es werden nur die Bänder B04, B03, B02 (10 m) und die Szenenklassifikation SCL (20 m) per HTTP-Range gelesen.
- **Maske:** Gültig sind die SCL-Klassen 4 (Vegetation), 5 (Boden), 6 (Wasser) und 11 (Schnee); Wolken, Schatten, Zirrus, dunkle und unklassifizierte Pixel zählen nicht. Die Maske wird um 40 m verkleinert, außerdem fallen Pixel mit sehr hohem Blauanteil (Dunst) weg, außer Schnee.
- **Robuster Wert statt letzter Wert:** Je Pixel hält der Bestand die letzten 5 gültigen Beobachtungen. Ausgegeben wird aus den Beobachtungen der letzten 45 Tage vor der jüngsten ein Wert mit mittlerer Helligkeit; Ausreißer (zu blau, zu hell, zu dunkel) gegen den Median werden verworfen. So bleibt das Bild aktuell, Dunst und dünne Wolken fallen heraus.
- **Lücken:** Ältere Beobachtungen füllen Pixel, die jüngst nie klar waren; Reste werden aus den Nachbarpixeln aufgefüllt.
- **Nähte:** Eine Szene wird in der Helligkeit an den Bestand angeglichen, die Ränder zwischen Aufnahmetagen sind weich übergeblendet.

## Hinweise

- Das Bild ist eine Mischung verschiedener Tage (Datum je Pixel im Bestand); dichter Bodennebel über Wochen kann trotz Filter stehen bleiben.
- 10 m Quellauflösung: z13 ist die letzte Stufe mit echtem Detailgewinn.

## Attribution

Pflicht nach den Copernicus-Nutzungsbedingungen (frei, vollständig und offen) für bearbeitete Daten:

<Code lang="html" code={"Contains modified Copernicus Sentinel data 2026"} />

Die Jahreszahl nennt das Jahr der Aufnahmen; MapLibre übernimmt den Text aus TileJSON oder PMTiles. Mehr: [Lizenz](/de/reference/attribution/).
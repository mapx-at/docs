# Sentinel-2 aktuell


Ein Satellitenbild Österreichs, das sich täglich fortschreibt: Für jedes Pixel gilt ein **robuster Wert aus den jüngsten wolken- und schattenfreien Sentinel-2-Aufnahmen** (Rolling Best-Pixel-Composite). Auflösung 10 m, Web-Mercator, WebP, 512 px, z5 bis z13 (darüber überzoomt der Client). Das Aufnahmedatum jedes Pixels wird im Bestand mitgeführt.



## Einbinden

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      center: [15.334, 48.227], zoom: 11,
      style: {
        version: 8,
        sources: { s2: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/sentinel2.json", tileSize: 512 } },
        layers: [{ id: "s2", type: "raster", source: "s2" }]
      }
    });
    ```
    TileJSON liefert Zoombereich, Grenzen und die Attribution. Der fertige Stil `satellit-aktuell` (Sentinel-2 mit Straßen und Namen) steht unter `/v1/styles/satellit-aktuell`.
    **Leaflet**

    ```js
    L.tileLayer("https://tiles.mapx.at/v1/tiles/sentinel2/{z}/{x}/{y}.webp", {
      tileSize: 512, zoomOffset: -1, minZoom: 5, maxNativeZoom: 13, maxZoom: 18, crossOrigin: true,
      attribution: "Contains modified Copernicus Sentinel data 2026"
    }).addTo(map);
    ```
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/sentinel2.json
    curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/sentinel2/13/4444/2839.webp
    ```
  
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

## Index- und Datumsebenen

Aus demselben Composite entstehen vier weitere feste Tilesets. Wo kein Wert vorliegt, sind sie durchsichtig und eignen sich als Überlagerung auf `sentinel2`, einem Luftbild oder einer Karte. Alle: WebP, 512 px, z5 bis z13, gleiche Attribution, täglich gemeinsam mit `sentinel2` ersetzt. In `/v1/tiles` erscheinen sie erst, wenn die Datei existiert. Das TileJSON enthält Wertebereich und Farbrampe in `mapx:legend`.

| Tileset | Zeigt | Wert und Farbe |
|---|---|---|
| `sentinel2-datum` | Aufnahmedatum je Pixel | Alter in Tagen vor `mapx:stand`: 0 grün, 20 gelb, 45 orange, 90 rot, 180 dunkelrot |
| `sentinel2-ndvi` | Vegetation, (NIR - Rot) / (NIR + Rot) | -0,1 grau, 0,05 kahl, 0,2 sandfarben, 0,4 hellgrün, 0,85 dunkelgrün |
| `sentinel2-ndwi` | Wasser, (Grün - NIR) / (Grün + NIR) | nur Werte über 0: hellblau (0) bis dunkelblau (0,7) |
| `sentinel2-ndsi` | Schnee, (Grün - SWIR) / (Grün + SWIR) | nur Werte ab 0,4: hellblau bis weiß (0,8); Wasser fällt über eine NIR-Schwelle heraus |

```js
map.addSource("ndvi", { type: "raster", url: "https://tiles.mapx.at/v1/tiles/sentinel2-ndvi.json", tileSize: 512 });
map.addLayer({ id: "ndvi", type: "raster", source: "ndvi", paint: { "raster-opacity": 0.8 } });
```

Im mapx-Viewer sind sie die Überlagerungen „Aufnahmedatum“, „Vegetation (NDVI)“, „Wasser (NDWI)“ und „Schnee (NDSI)“ in der Gruppe Satellit, mit Legende und Deckkraft-Regler.

## Hinweise

- Das Bild ist eine Mischung verschiedener Tage (Datum je Pixel im Bestand); dichter Bodennebel über Wochen kann trotz Filter stehen bleiben.
- 10 m Quellauflösung: z13 ist die letzte Stufe mit echtem Detailgewinn.

## Attribution

Pflicht nach den Copernicus-Nutzungsbedingungen (frei, vollständig und offen) für bearbeitete Daten:

```html
Contains modified Copernicus Sentinel data 2026
```

Die Jahreszahl nennt das Jahr der Aufnahmen; MapLibre übernimmt den Text aus TileJSON oder PMTiles. Mehr: [Lizenz](/de/reference/attribution/).


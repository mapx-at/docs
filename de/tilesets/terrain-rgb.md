# DTM-Geländemodell (Terrain-RGB)

Source: https://docs.mapx.at/de/tilesets/terrain-rgb/

> Digitales Geländemodell Österreich als Terrain-RGB-Kacheln (Mapbox-Kodierung, 0,1 m), z5 bis z16, aus dem BEV ALS-DTM 1 m.

Das DTM ist ein Höhenkachelsatz für ganz Österreich: Geländehöhe ohne Gebäude und Bewuchs, kodiert als Farbe (Terrain-RGB). Es ist direkt für `raster-dem`, Schummerung und 3D-Gelände in MapLibre, Mapbox GL und kompatiblen Clients verwendbar.

**Nutzungen**

- **Wander- und Outdoorkarten**: Schummerung als Kartenhintergrund bis z16
- **3D-Gelände**: Gelände und Flugansichten mit Orthophoto (https://docs.mapx.at/de/guides/3d-terrain/)
- **Höhenprofile**: Aufstieg und Gefälle entlang einer Route (https://docs.mapx.at/de/guides/elevation-profile/)
- **Hang- und Expositionsanalyse**: Neigung und Ausrichtung aus den Höhenwerten
- **QGIS und Analyse**: Höhen als Raster im Desktop-GIS (https://docs.mapx.at/de/guides/qgis/)

## Kennwerte

**Tileset `terrain-rgb`**

| Kennwert | Wert |
|---|---|
| Stichtag | `15.09.2025` |
| Format | `webp, Kodierung mapbox, 512 px` |
| Zoomstufen | `z5–z16` |
| Gebiet (WGS84) | `9,5306 / 46,3697 / 17,1661 / 49,0235 (W/S/O/N)` |
| Vertikale Auflösung | `0,1 m (Terrain-RGB)` |
| Höhengenauigkeit (Quelle) | `± 0,50 m laut BEV, im Hochgebirge größer` |
| Höhenbezug | `EVRF2000 Austria, orthometrisch (EPSG:9274)` |
| Lizenz | `CC BY 4.0` |

## Live-Karte

Der Klick liest die Kachel der aktuellen Zoomstufe (maximal z16) per Range-Request, dekodiert das Pixel am Klickpunkt mit der
Formel unter „Kodierung“ und zeigt Höhe und RGB-Wert. Bei eingeschaltetem 3D steht zusätzlich der Wert von `queryTerrainElevation` dabei.

## Einbinden

**MapLibre**

```js
import * as maplibregl from "maplibre-gl";

maplibregl.addProtocol("pmtiles", new Protocol().tile);
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  center: [12.694, 47.074], zoom: 11, pitch: 60,
  style: {
    version: 8,
    sources: { dem: { type: "raster-dem", url: "pmtiles://@@url:terrain-rgb@@?access_token=DEIN_TOKEN", tileSize: 512, encoding: "mapbox", maxzoom: 16 } },
    terrain: { source: "dem", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dem" }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
map.on("click", (e) => console.log(map.queryTerrainElevation(e.lngLat)));
```

  

**QGIS**

1. Browser, Rechtsklick auf **XYZ Tiles**, **Neue Verbindung**.
2. URL `https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN`, Min. Zoom 5, Max. Zoom 16, Kachelauflösung 512.
3. Der Layer zeigt die Rohkacheln (kodierte Farbe). Höhen ergeben sich aus der Formel in „Kodierung“ (`(R*65536 + G*256 + B)*0,1 - 10000`), etwa im Raster-Rechner auf den drei Bändern.

  

**Python**

```python
# pip install pillow numpy requests
import io
import numpy as np
import requests
from PIL import Image

z, x, y = 12, 2192, 1439                   # Großglockner
r = requests.get(f"https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp",
                 headers={"Authorization": "Bearer DEIN_TOKEN"}, timeout=30)
r.raise_for_status()
tile = r.content
rgb = np.asarray(Image.open(io.BytesIO(tile)).convert("RGB")).astype(np.uint32)
hoehe = -10000 + (rgb[..., 0] * 65536 + rgb[..., 1] * 256 + rgb[..., 2]) * 0.1
print(hoehe.shape, hoehe.min(), hoehe.max())
```

  

**curl**

```bash
# eine Kachel über den XYZ-Endpunkt
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/terrain-rgb/12/2192/1439.webp
```

## Kodierung

```
höhe_m = -10000 + (R · 65536 + G · 256 + B) · 0,1
```

- Wertebereich theoretisch −10000 m bis +1 667 721,5 m in Schritten von 0,1 m; in Österreich liegen die Werte zwischen rund 115 m (Neusiedler See) und rund 3 798 m (Großglockner).
- Gerundet wird beim Schreiben auf 0,1 m (Rundung zur geraden Zahl bei genau .5), Werte außerhalb des Bereichs werden begrenzt.
- WebP **verlustfrei**: verlustbehaftete Kompression würde die kodierten Höhen direkt verfälschen. Keine Alpha-Information nutzen, RGB-Bänder unverändert lesen (kein Farbmanagement, kein Premultiplied Alpha).
- 512 × 512 Pixel, **ohne Rand** (kein 1-px-Skirt). In MapLibre `tileSize: 512` und `encoding: "mapbox"` angeben.

Ausführlich: [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md).

## Vertikale Auflösung und Genauigkeit

| | Wert | Bedeutung |
|---|---|---|
| Vertikale Auflösung | **0,1 m** | kleinster Höhenschritt der Kodierung |
| Abweichung zur Quelle | **≤ 0,05 m** | Rundung beim Kodieren, sonst keine Veränderung (verlustfrei) |
| Höhengenauigkeit der Quelle | **± 0,50 m** | Angabe des BEV für das ALS-DTM; im Hochgebirge sind größere Abweichungen möglich |
| Höhenbezug | EVRF2000 Austria | orthometrische Höhen (EPSG:9274), wie in der Quelle |

Die Kodierung ist also fünfmal feiner als die Genauigkeit der Messung: Sie begrenzt die Qualität nicht.

In den Übersichtsstufen z15 bis z5 ist jedes Pixel der Mittelwert der feineren Stufe. Die vertikale Auflösung bleibt 0,1 m, kleine Spitzen und Gipfel werden aber geglättet. Für exakte Punkthöhen z16 lesen oder die [Höhen-API](https://docs.mapx.at/de/api/elevation.md) nutzen.

## Horizontale Auflösung je Zoom

Bei 47,5° N, 512-px-Kacheln, Web-Mercator:

| Zoom | m pro Pixel | Kantenlänge Kachel (km) | Anmerkung |
|---|---|---|---|
| z5 | 1.652 | 846 |  |
| z6 | 826 | 423 |  |
| z7 | 413 | 212 |  |
| z8 | 207 | 106 |  |
| z9 | 103 | 53 |  |
| z10 | 52 | 26 |  |
| z11 | 26 | 13 |  |
| z12 | 13 | 6,61 |  |
| z13 | 6,46 | 3,30 |  |
| z14 | 3,23 | 1,65 |  |
| z15 | 1,61 | 0,83 |  |
| z16 | 0,81 | 0,41 | höchste gespeicherte Stufe (≈ 1-m-Ursprung) |
| z17 | 0,40 | 0,21 | Overzoom (Client) |

Die Daten stammen aus einem 1-m-Raster. z16 (≈ 0,8 m pro Pixel) ist die feinste Stufe und erhält die Auflösung der Quelle.
Über z16 gibt es keine Kacheln: MapLibre zoomt ab z17 die z16-Kachel auf (`maxzoom: 16` setzen).

## Quelle und Aktualität

Das DTM entsteht aus dem ALS-DTM 1 m des BEV (Airborne Laserscanning, nur Gelände). Der Stichtag steht in den Kennwerten und im Dateinamen; eine Datei ändert sich nie, ein neuer Stand ist eine neue Datei.

- Quelle: BEV (Bundesamt für Eich- und Vermessungswesen), Stichtag 15.09.2025.

## Hinweise

- Quantisierung 0,1 m. Die Abweichung zur Quelle liegt im Rundungsbereich (höchstens 0,05 m).
- Höhengenauigkeit laut BEV ± 0,50 m, Details unter [Vertikale Auflösung und Genauigkeit](#vertikale-auflösung-und-genauigkeit).
- Das DTM zeigt das Gelände ohne Gebäude, Bäume und Brücken. Für die Oberfläche gibt es das [DSM](https://docs.mapx.at/de/tilesets/dsm.md).
- **Kein Messwert außerhalb der Daten.** Das Gebiet des Kachelsatzes ist das Rechteck 9,53–17,17° O, 46,37–49,02° N. Pixel ohne Quelldaten (außerhalb Österreichs, Lücken) sind eine glatte Fortsetzung des Geländes, keine Messung.

## Attribution

Pflicht nach CC BY 4.0. Text:

```
Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0
```

HTML (wie in den PMTiles-Metadaten, MapLibre übernimmt es automatisch aus dem Tileset):

```html
Relief: <a href="https://data.bev.gv.at/geonetwork/srv/ger/catalog.search#/metadata/5ce253fc-b7c5-4362-97af-6556c18a45d9">BEV – ALS-DTM 1 m</a> (Stichtag 15.09.2025), CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

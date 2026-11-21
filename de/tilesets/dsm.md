# DSM-Oberflächenmodell (Kachelsatz)

Source: https://docs.mapx.at/de/tilesets/dsm/

> Digitales Oberflächenmodell Österreich mit Gebäuden und Bewuchs als Terrain-RGB (0,1 m), z5 bis z16, aus dem BEV ALS-DOM 1 m.

Das DSM ist das Gegenstück zum [DTM](https://docs.mapx.at/de/tilesets/terrain-rgb.md): Es zeigt die Oberfläche mit Gebäuden, Bäumen und Brücken. Kodierung, Kachelgröße und Zoomstufen sind identisch, alle Clients und Formeln des DTM gelten unverändert.

**Nutzungen**

- **Gebäude- und Bewuchshöhe**: nDSM aus DSM minus DTM im Client
- **Solarplanung**: Dach- und Verschattungshöhen als Grundlage
- **Sichtbarkeit und Verschattung**: Oberfläche inklusive Bebauung und Wald
- **3D-Stadt- und Landschaftsansicht**: Oberfläche als Terrain im Browser (https://docs.mapx.at/de/guides/3d-terrain/)
- **Höhenprofile**: Profil über der Oberfläche (https://docs.mapx.at/de/guides/elevation-profile/)

## Kennwerte

**Kachelsatz `dsm`**

| Kennwert | Wert |
|---|---|
| Stichtag | `15.09.2025` |
| Format | `webp, Kodierung mapbox, 512 px` |
| Zoomstufen | `z5–z16` |
| Gebiet (WGS84) | `9,5306 / 46,3697 / 17,1661 / 49,0235 (W/S/O/N)` |
| Vertikale Auflösung | `0,1 m (Terrain-RGB)` |
| Lizenz | `CC BY 4.0` |

## Live-Karte

Der Klick dekodiert das Pixel der Kachel (Formel unter [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md)).

## Einbinden

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { dsm: { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/dsm.json", tileSize: 512, encoding: "mapbox" } },
    terrain: { source: "dsm", exaggeration: 1.2 },
    layers: [{ id: "hill", type: "hillshade", source: "dsm" }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**Leaflet**

```js
L.tileLayer("https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "mapx.at; BEV, CC BY 4.0"
}).addTo(map);
```

  

**QGIS**

XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/dsm/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN`.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/dsm.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/dsm/14/8860/5730.webp
```

Höhe per HTTP: [Höhe und Profil](https://docs.mapx.at/de/api/elevation.md) mit `model=dsm`.

## Quelle und Aktualität

Das DSM entsteht aus dem ALS-DOM 1 m des BEV (Airborne Laserscanning, Oberfläche). Der Stichtag steht in den Kennwerten und im Dateinamen; eine Datei ändert sich nie, ein neuer Stand ist eine neue Datei.

- Quelle: BEV ALS-DOM 1 m, Stichtag 15.09.2025.
- Gleiche Kodierung wie das DTM: Mapbox-Terrain-RGB mit 0,1 m Quantisierung, 512 px, WebP verlustfrei.

## Hinweise

- Quantisierung 0,1 m, maximal z16 (≈ 0,8 m pro Pixel bei 48° N), darüber zoomt der Client auf.
- Die Genauigkeit entspricht der amtlichen Quelle des BEV.
- Gebäude und Bewuchs entsprechen dem Zeitpunkt der Befliegung, nicht dem Stichtag.
- Außerhalb der Daten ist der Wert eine glatte Fortsetzung, keine Messung (wie beim DTM).
- Gebäude- und Bewuchshöhe (nDSM) ergeben sich im Client aus DSM minus DTM.

## Quellenangabe

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus dem TileJSON:

```html
mapx.at; Relief: <a href="https://data.bev.gv.at/geonetwork/srv/ger/catalog.search#/metadata/cced2282-cfc5-42eb-9acb-b1c799522fcd">BEV – ALS-DOM 1 m</a> (Stichtag 15.09.2025), CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

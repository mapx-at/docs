# Orthophoto-Tileset: Luftbild Österreich

Source: https://docs.mapx.at/de/tilesets/orthophoto/

> Luftbild Österreich des BEV als WebP-Kacheln, z5 bis z18, 20 cm, mit eindeutiger Flugblock-Auswahl je Pixel.

Das Orthophoto des BEV, neu gekachelt in Web-Mercator bis z18 (20 cm, die native Auflösung der Quelle). Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](https://docs.mapx.at/de/tilesets/orthophoto-flight-blocks.md).

**Nutzungen**

- **Luftbild-Hintergrund**: Satellitenkarten-Ersatz für Web und Apps
- **3D-Gelände**: Orthophoto auf dem DTM (https://docs.mapx.at/de/guides/3d-terrain/)
- **Planung und Dokumentation**: Zustand vor Ort in 20 cm
- **Grundstücke prüfen**: Kataster über dem Luftbild
- **QGIS**: Luftbild als XYZ- oder PMTiles-Layer (https://docs.mapx.at/de/guides/qgis/)

## Kennwerte

**Tileset `dop-ortho`**

| Kennwert | Wert |
|---|---|
| Stichtag | `03.10.2025` |
| Format | `webp` |
| Zoomstufen | `z5–z18` |
| Gebiet (WGS84) | `9,4500 / 46,3000 / 17,2500 / 49,1000 (W/S/O/N)` |
| Lizenz | `CC BY 4.0` |

## Live-Karte

Klick zeigt den Flugblock am Punkt (Operat, Flugzeitraum).

## Einbinden

**MapLibre**

```js
new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { ortho: { type: "raster", url: "https://tiles.mapx.at/v1/tiles/orthophoto.json", tileSize: 512 } },
    layers: [{ id: "ortho", type: "raster", source: "ortho" }]
  }
});
```
TileJSON liefert Zoombereich, Grenzen und Attribution. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen. Direkt per PMTiles geht `url: "pmtiles://https://tiles.mapx.at/dop-ortho-20251003-272a54b3.pmtiles?access_token=DEIN_TOKEN"` (Protokoll wie im [PMTiles-Guide](https://docs.mapx.at/de/guides/self-host-pmtiles/)).

  

**Leaflet**

```js
L.tileLayer("https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN", {
  tileSize: 512, zoomOffset: -1, crossOrigin: true,
  attribution: "BEV, CC BY 4.0"
}).addTo(map);
```

  

**QGIS**

XYZ Tiles → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN`.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.webp https://tiles.mapx.at/v1/tiles/orthophoto/14/8860/5730.webp
```

## Quelle und Aktualität

- Quelle: BEV **Digitales Orthophoto (DOP) RGB**, 20 cm, Aufnahmen 2022 bis 2025.
- **Ein Flug je Pixel:** Wo sich Befliegungen überlappen, gewinnt die mit dem späteren Ende des Flugzeitraums. Welcher Flug ein Pixel liefert, zeigt das Tileset [Flugblöcke](https://docs.mapx.at/de/tilesets/orthophoto-flight-blocks.md).

## Hinweise

- Ganz Österreich ist bedeckt.
- Zwischen Bildflügen sind Farbnähte sichtbar, es gibt keine Farbangleichung.
- An den Grenzen zu DE, CH, LI, IT, SI, CZ und SK reicht das Bild teils mehr als 3 km ins Ausland; an der Grenze zu Ungarn endet es an der Grenze.
- Unter z18 mischen sich an Blockgrenzen Pixel beider Flüge; eindeutig ist die Zuordnung nur in z18.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

```html
Orthophoto: BEV (CC BY 4.0)
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

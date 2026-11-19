# Orthophoto-Flugblöcke (Tileset)

Source: https://docs.mapx.at/de/tilesets/orthophoto-flight-blocks/

> Welcher Bildflug liefert welches Orthophoto-Pixel: Gewinnerflächen der BEV-Operate mit Flugzeitraum, als Vektorkacheln z6 bis z17.

Zu jedem Pixel des [Orthophotos](https://docs.mapx.at/de/tilesets/orthophoto.md) die Fläche des Bildflugs (Operats), aus dem es stammt, mit Flugzeitraum.

**Nutzungen**

- **Aufnahmezeitpunkt anzeigen**: „Aufnahme von“ in der Karte
- **Aktualität einschätzen**: Alter des Luftbilds an einem Ort
- **Bildwechsel erklären**: Farbnähte zwischen Flügen nachvollziehen
- **Qualitätsprüfung**: Welcher Flug liefert welches Gebiet

## Kennwerte

**Tileset `orthophoto-flight-blocks`**

| Kennwert | Wert |
|---|---|
| Stichtag | `03.10.2025` |
| Format | `mvt` |
| Zoomstufen | `z6–z17` |
| Gebiet (WGS84) | `9,4500 / 46,3000 / 17,2500 / 49,1000 (W/S/O/N)` |
| Lizenz | `CC BY 4.0` |

## Live-Karte

## Einbinden

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json" } },
    layers: [{ id: "orthophoto-flight-blocks", type: "line", source: "v", "source-layer": "dop_flugblock", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Attribution. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf
```

## Layer-Referenz

### orthophoto-flight-blocks

Geometrie: **Fläche**, Zoom: **z6–17**, Layer: `dop_flugblock`. Befliegungsblöcke der Orthofotos mit Aufnahmedatum.

| Feld | Typ | Beschreibung |
|---|---|---|
| `archivnr` | `number` | Archivnummer der Befliegung. |
| `fid` | `number` | Laufende Nummer des Flugblocks. |
| `flug_bis` | `string` | Ende der Befliegung. |
| `flug_von` | `string` | Beginn der Befliegung. |
| `flugblock_id` | `string` | Kennung des Flugblocks. |
| `flugdatum` | `string` | Flugtag (JJJJ-MM-TT). |
| `flugjahr` | `number` | Jahr der Befliegung. |
| `mosaik_url` | `string` | Adresse des zugehörigen Orthofoto-Mosaiks. |
| `operat` | `string` | Bezeichnung des Operats (Auftrags). |
| `rang` | `number` | Vorrang des Flugblocks bei Überlappung (0 = höchster Vorrang). |
| `stichtag` | `string` | Datenstand. |

## Quelle und Aktualität

- Quelle: BEV „Aktualität Orthophoto Operate Farbe und Infrarot“.
- Die Flächen zeigen, wo ein Operat im Orthophoto **tatsächlich** zu sehen ist, nicht die ursprünglichen Umrisse. Ein Operat kann mehrere Flächen haben.

## Hinweise

- Ein Flugblock ist ein Operat, kein einzelner Flugtag: Es gibt kein Tagesdatum, nur den Zeitraum des Operats; ein Operat kann bis zu drei Monate dauern.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

```html
BEV, CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

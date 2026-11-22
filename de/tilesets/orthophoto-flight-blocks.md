# Orthofoto-Flugblöcke (Tileset)

Source: https://docs.mapx.at/de/tilesets/orthophoto-flight-blocks/

> Welcher Bildflug liefert welches Orthofoto-Pixel: Gewinnerflächen der BEV-Operate mit Flugzeitraum, als Vektorkacheln z6 bis z17.

Zu jedem Pixel des [Orthofotos](https://docs.mapx.at/de/tilesets/orthophoto.md) die Fläche des Bildflugs (Operats), aus dem es stammt, mit Flugzeitraum.

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
    sources: { v: { type: "vector", tiles: ["https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf"], minzoom: 6, maxzoom: 17, attribution: "mapx.at | BEV, CC BY 4.0" } },
    layers: [{ id: "orthophoto-flight-blocks", type: "line", source: "v", "source-layer": "flight_block_a", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
Die Quelle nennt Kachel-Adresse, Zoombereich und Quellenangabe selbst; die TileJSON-Adresse `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json` liefert Zoombereich und Grenzen, aber nur `mapx.at` als Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf
```

## Ebenen-Referenz

### flight_block_a

Geometrie: **Fläche**, Zoom: **z6–17**, Ebene: `flight_block_a`. Befliegungsblöcke der Orthofotos mit Aufnahmedatum.

| Feld | Typ | Beschreibung |
|---|---|---|
| `archive_number` | `number` | Archivnummer der Befliegung. |
| `block_id` | `string` | Kennung des Flugblocks. |
| `fid` | `number` | Laufende Nummer des Flugblocks. |
| `flight_date` | `string` | Ende des Befliegungszeitraums (JJJJ-MM-TT), bestimmt den Vorrang. |
| `flight_from` | `string` | Beginn der Befliegung. |
| `flight_to` | `string` | Ende der Befliegung. |
| `flight_year` | `number` | Jahr der Befliegung. |
| `mosaic_url` | `string` | Adresse des zugehörigen Orthofoto-Mosaiks. |
| `operat` | `string` | Bezeichnung des Operats (Auftrags). |
| `rank` | `number` | Vorrang des Flugblocks bei Überlappung (0 = höchster Vorrang). |
| `reference_date` | `string` | Datenstand. |

## Quelle und Aktualität

- Quelle: BEV „Aktualität Orthofoto Operate Farbe und Infrarot“.
- Die Flächen zeigen, wo ein Operat im Orthofoto **tatsächlich** zu sehen ist, nicht die ursprünglichen Umrisse. Ein Operat kann mehrere Flächen haben.

## Hinweise

- Ein Flugblock ist ein Operat, kein einzelner Flugtag: Es gibt kein Tagesdatum, nur den Zeitraum des Operats; ein Operat kann bis zu drei Monate dauern.

## Quellenangabe

Pflicht nach CC BY 4.0. Das TileJSON nennt nur `mapx.at`, die Quelle selbst ergänzen (wie im MapLibre-Beispiel oben):

```html
mapx.at | Orthophoto: BEV (CC BY 4.0)
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

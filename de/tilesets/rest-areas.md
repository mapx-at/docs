# Rastanlagen-Tileset (ASFINAG)

Source: https://docs.mapx.at/de/tilesets/rest-areas/

> Rastanlagen und Lkw-Parkplätze der ASFINAG als Vektorkacheln (Tileset rest-areas) und als Suchkategorie rest_area in der mapx-Suche.

Rast-, Pendler- und Lkw-Parkplätze an Autobahnen und Schnellstraßen aus den Verkehrsdaten der ASFINAG, 314 Standorte.

> **Achtung:** **Quellenangabe**
> 
> mapx.at | Datenquelle: ASFINAG, Rast-, Pendler- und Lkw-Parkplätze, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Bearbeitung: Auswahl und Umformung der Felder.

## Kennwerte

**Tileset `rest-areas`**

| Kennwert | Wert |
|---|---|
| Stichtag | `20.11.2026` |
| Format | `mvt` |
| Zoomstufen | `z7–z14` |
| Gebiet (WGS84) | `9,6258 / 46,5314 / 17,1018 / 48,7061 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/rest-areas.json" } },
    layers: [{ id: "rest_area_p", type: "circle", source: "v", "source-layer": "rest_area_p", paint: { "circle-color": "#c0392b", "circle-radius": 5, "circle-stroke-color": "#fff", "circle-stroke-width": 1 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/rest-areas/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/rest-areas.json
```

## Ebenen-Referenz

### rest_area_p

Geometrie: **Punkt**, Zoom: **z7–14**, Ebene: `rest_area_p`. Rast-, Pendler- und Lkw-Parkplätze der ASFINAG.

| Feld | Typ | Beschreibung |
|---|---|---|
| `car` | `number` | Anzahl der Pkw-Stellplätze. |
| `free` | `boolean` | true bei kostenfreier Nutzung. |
| `id` | `string` | Kennung des Standorts (ASFINAG), zum Beispiel PARK000109. |
| `lorry` | `number` | Anzahl der Lkw-Stellplätze. |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt werden soll. |
| `name` | `string` | Name des Standorts. |
| `road` | `string` | Straße, an der der Standort liegt, zum Beispiel A01. |
| `usage` | `string` | Art des Standorts. Werte: `truckParking` = Lkw-Parkplatz; `serviceArea` = Raststation mit Serviceeinrichtungen; `restArea` = Rastplatz |

## Quelle und Aktualität

- Quelle: ASFINAG, Verkehrsdaten (Rast-, Pendler- und Lkw-Parkplätze). Der Stichtag steht in den Kennwerten.

## Hinweise

- Die Belegung ist nicht enthalten.
- Die Stellplatzzahlen `car` und `lorry` sind die Angaben der ASFINAG, `0` heißt keine Plätze dieser Art.

## Weiterverwendung

| | |
|---|---|
| Stil | Gruppe `rest_areas`, standardmäßig aus: Punkt, ab z12 Name; Anlagen mit Lkw-Plätzen dunkler |
| Suche | Kategorie `rest_area` in [Search Box](https://docs.mapx.at/de/api/search-box.md) und Geocoding |
| Abfrage | [Tilequery](https://docs.mapx.at/de/api/tilequery.md) mit `rest-areas` als Tileset |

## Quellenangabe

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus dem TileJSON:

```html
mapx.at | <a href="https://www.asfinag.at" target="_blank" rel="noopener">ASFINAG</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

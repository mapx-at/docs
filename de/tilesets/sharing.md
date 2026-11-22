# Tileset Sharing-Standorte (ÖBB)

Source: https://docs.mapx.at/de/tilesets/sharing/

> Sharing-Standorte (Lastenräder, Fahrräder) der ÖBB als Vektorkacheln (Tileset sharing) und als Suchkategorie vehicle_sharing in der mapx-Such-API.

Standorte von Sharing-Stationen aus dem Datensatz „Shared-Mobility Standorte 2026“ der ÖBB-Personenverkehr AG. Der Datensatz enthält nur Standorte, die die ÖBB selbst betreibt und besitzt; derzeit sieben Stationen in Korneuburg und Leoben.

> **Achtung:** **Quellenangabe**
> 
> mapx.at; Datenquelle: ÖBB-Personenverkehr AG, Shared-Mobility Standorte, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de).

## Kennwerte

**Tileset `sharing`**

| Kennwert | Wert |
|---|---|
| Stichtag | `20.11.2026` |
| Format | `mvt` |
| Zoomstufen | `z8–z14` |
| Gebiet (WGS84) | `15,0924 / 47,3621 / 16,3339 / 48,3443 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/sharing.json" } },
    layers: [{ id: "sharing_p", type: "circle", source: "v", "source-layer": "sharing_p", paint: { "circle-color": "#1b7f5a", "circle-radius": 5, "circle-stroke-color": "#fff", "circle-stroke-width": 1 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/sharing/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/sharing.json
```

## Ebenen-Referenz

### sharing_p

Geometrie: **Punkt**, Zoom: **z8–14**, Ebene: `sharing_p`. Sharing-Standorte der ÖBB-Personenverkehr AG.

| Feld | Typ | Beschreibung |
|---|---|---|
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt werden soll. |
| `municipality` | `string` | Gemeinde. |
| `name` | `string` | Name des Standorts. |
| `operator` | `string` | Betreiber, zum Beispiel ÖBB-PV. |
| `vehicle` | `string` | Fahrzeugart. Werte: `cargo_bike` = Lastenrad; `bike` = Fahrrad |

## Quelle und Aktualität

- Quelle: ÖBB-Personenverkehr AG, Datensatz „Shared-Mobility Standorte 2026“ (data.oebb.at). Der Stichtag steht in den Kennwerten.

## Hinweise

- Die Quelldatei hat einen Standort der Leobener Station „Waasenbrücke“ mit einem Breitengrad rund 40 km neben der Stadt; er wurde nicht übernommen.

## Weiterverwendung

| | |
|---|---|
| Stil | Gruppe `sharing`, standardmäßig aus: Punkt ab z10, Name ab z14 |
| Suche | Kategorie `vehicle_sharing` in [Search Box](https://docs.mapx.at/de/api/search-box.md) und Geocoding |
| Abfrage | [Tilequery](https://docs.mapx.at/de/api/tilequery.md) mit `sharing` als Tileset |

## Quellenangabe

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus dem TileJSON:

```html
mapx.at; <a href="https://data.oebb.at/de/datensaetze~shared-mobility-standorte~" target="_blank" rel="noopener">ÖBB-Personenverkehr AG – Shared-Mobility Standorte</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

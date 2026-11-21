# Vektor-Basiskarte (Tileset streets)

Source: https://docs.mapx.at/de/tilesets/streets/

> Vektor-Basiskarte Österreich aus offenen Behördendaten: Nutzung, Gebäude, Straßen, Bahn, Gewässer, Grenzen, Namen, Hausnummern. z0 bis z16.

Die Basiskarte vereint die offenen Daten von BEV, GIP, Umweltbundesamt und BMLUK in 19 Layern. Sie enthält keine OpenStreetMap-Daten. Höhenlinien und Relief sind eigene Produkte und lassen sich darüberlegen.

**Nutzungen**

- **Eigener Kartenstil**: Die Layer nach Wunsch stylen (https://docs.mapx.at/de/guides/style-the-basemap/)
- **Web-Karten und Apps**: Vektorkarte mit eigener Gestaltung bis z16
- **Planungs- und Navigationshintergrund**: Straßen, Bahn, Gewässer, Grenzen
- **Hausnummern und Namen**: Adressen, Orte, Berge und Gewässernamen
- **Überlagerung**: Höhenlinien und Relief darüber (https://docs.mapx.at/de/guides/overlay-contours/)

## Kennwerte

**Tileset `streets`**

| Kennwert | Wert |
|---|---|
| Stichtag | `01.04.2026` |
| Format | `mvt` |
| Zoomstufen | `z0–z16` |
| Gebiet (WGS84) | `8,4375 / 45,0890 / 19,6875 / 50,7365 (W/S/O/N)` |
| Lizenz | `CC BY 4.0` |

## Live-Karte

Die Farben der Live-Karte sind automatisch vergeben (Fläche, Linie, Punkt je Layer), kein fertiger Kartenstil. Klick zeigt die Attribute.

## Einbinden

**MapLibre**

```js
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
    layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Attribution. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/streets.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/streets/14/8860/5730.pbf
```

## Layer-Referenz

Layername endet auf `_a` (Fläche), `_l` (Linie) oder `_p` (Punkt). `min_zoom` am Objekt sagt, ab welcher Stufe es gedacht ist; `sort_rank` ordnet innerhalb des Layers.

| Layer | Geometrie | Zoom | Attribute | Inhalt |
|---|---|---|---|---|

Alle Felder und Werte jedes Layers: [Basiskarte: Layer-Referenz](https://docs.mapx.at/de/reference/style-spec.md).

## Quelle und Aktualität

| Thema | Quelle | Stand |
|---|---|---|
| Nutzung, Gebäude | BEV Digitale Katastralmappe (DKM), Höhen aus dem DLM | 01.04.2026 |
| Gebäudehöhen ohne DLM-Wert, Turmteile | BEV ALS-Oberflächen- und Geländemodell 1 m | 15.09.2025 |
| Gebäudeteile Wien | Stadt Wien – data.wien.gv.at, Baukörpermodell (CC BY 4.0) | 21.11.2026 |
| Straßen, Bahn | GIP-OGD (Graphenintegrationsplattform) | 30.04.2026 |
| Lifte, Stationen, POI | BEV DLM, Wien-OGD (nur Wien) | |
| Grenzen | BEV Verwaltungsgrenzen (VGD) | 01.10.2026 |
| Orte, Namen, Gipfel | BEV Geographische Namen | 23.03.2026 |
| Hausnummern | BEV Adressregister | 01.10.2026 |
| Gewässer | BMLUK Gesamtgewässernetz (GGN v19) | |
| Skigebiete | Umweltbundesamt | 2017 |
| Flugplätze | BEV DLM Verkehr (CC BY 4.0) | |

## Hinweise

- Nicht enthalten: Halte von Privatbahnen sowie U-Bahn- und Straßenbahnhaltestellen; kleine Flugfelder fehlen teils. Skigebiete haben den Stand 2017.
- Ausland: praktisch leer (nur Bodensee und Gewässer bis knapp hinter die Grenze).
- Glyphen und Schriften für Beschriftungen liefert mapx nicht; der Style braucht eigene.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

```html
Datenquellen: BEV (DKM, Verwaltungsgrenzen, Adressregister, Geographische Namen), GIP.at, BMLUK (GGN), Umweltbundesamt, Stadt Wien – data.wien.gv.at; CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

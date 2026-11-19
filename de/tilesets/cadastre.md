# Kataster-Tileset: Grundstücksgrenzen

Source: https://docs.mapx.at/de/tilesets/cadastre/

> Grundstücksgrenzen aus der Digitalen Katastralmappe des BEV als Vektorkacheln, z16, mit Katastralgemeinde und Grundstücksnummer.

Die Grenzen aller Grundstücke als Linien, mit Katastralgemeinde-Nummer und Grundstücksnummer, aus der Digitalen Katastralmappe des BEV. Das Tileset hat nur z16; Clients zoomen darüber auf.

**Nutzungen**

- **Grundstückssuche**: Grenzen und Nummern auf der Karte
- **Planung und Vorprüfung**: Grundstücksgrenzen über Orthophoto legen
- **Immobilien- und Standortkarten**: Parzellen als Kartenebene
- **Überlagerung**: Kataster über Basiskarte oder Luftbild (https://docs.mapx.at/de/guides/style-the-basemap/)
- **QGIS**: Vektorkacheln im Desktop-GIS (https://docs.mapx.at/de/guides/qgis/)

## Kennwerte

**Tileset `mapx-kataster`**

| Kennwert | Wert |
|---|---|
| Stichtag | `01.04.2026` |
| Format | `mvt` |
| Zoomstufen | `z16–z16` |
| Gebiet (WGS84) | `9,5251 / 46,3697 / 17,1661 / 49,0235 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" } },
    layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Attribution. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/cadastre/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/cadastre.json
curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/cadastre/14/8860/5730.pbf
```

Weil es nur z16 gibt, im Style `minzoom: 16` setzen; die Quelle mit `maxzoom: 16` überzoomt automatisch.

## Layer-Referenz

Die Linien sind Grenzen, keine Flächen: Ein Klick trifft die Grenzlinie und liefert das Grundstück, zu dem sie gehört. Eigentümer und Fläche sind nicht enthalten.

### parcel_l

Geometrie: **Linie**, Zoom: **z16–16**, Layer: `parcel_l`. Grundstücksgrenzen (Linien).

| Feld | Typ | Beschreibung |
|---|---|---|
| `gstnr` | `string` | Grundstücksnummer. |
| `kgnr` | `number` | Nummer der Katastralgemeinde. |

## Quelle und Aktualität

- Quelle: BEV **Digitale Katastralmappe (DKM)**, Stichtag 01.04.2026.

## Hinweise

> **Achtung:** Die Darstellung ist nicht rechtsverbindlich. Für Grenzfragen gilt der Kataster beim Vermessungsamt.

- Die Abbildung der Grenzen ist nicht rechtsverbindlich, maßgeblich ist der Kataster beim Vermessungsamt.
- Unter z16 gibt es keine Kacheln.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

```html
Datenquelle: BEV – DKM, CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

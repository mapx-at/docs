# Radrouten-Kachelsatz (GIP.at)

Source: https://docs.mapx.at/de/tilesets/bike-routes/

> Beschilderte Radrouten und MTB-Strecken in Österreich aus GIP.at, inklusive EuroVelo, als Vektorkacheln (Kachelsatz bike-routes) für MapLibre.

Radrouten und Mountainbike-Strecken aus dem offenen Datensatz der Graphenintegrationsplattform (GIP.at, GIPOGD), inklusive der EuroVelo-Routen. Im Stil `streets` stecken sie in der Ebenengruppe `bike_routes` (standardmäßig aus); siehe [Tourismus](https://docs.mapx.at/de/use-cases/tourism.md).

## Kennwerte

**Kachelsatz `bike-routes`**

| Kennwert | Wert |
|---|---|
| Stichtag | `08.07.2026` |
| Format | `mvt` |
| Zoomstufen | `z5–z14` |
| Gebiet (WGS84) | `9,4980 / 46,2993 / 17,1565 / 49,0141 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/bike-routes.json" } },
    layers: [{ id: "bike_routes", type: "line", source: "v", "source-layer": "bike_routes", paint: { "line-color": "#b5651d", "line-width": 2 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/bike-routes/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/bike-routes.json
```

## Ebenen-Referenz

### bike_routes

Geometrie: **Linie**, Zoom: **z5–14**, Ebene: `bike_routes`. Beschilderte Radrouten und Mountainbike-Strecken, EuroVelo eingeschlossen.

| Feld | Typ | Beschreibung |
|---|---|---|
| `difficulty` | `string` | Schwierigkeit, nur bei Mountainbike-Strecken und nicht bei allen. Werte: `easy` = leicht; `medium` = mittel; `hard` = schwer |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt werden soll. |
| `name` | `string` | Name der Route. |
| `network` | `string` | Bedeutung des Netzes. Werte: `international` = Internationale Route (EuroVelo); `national` = Nationale Route; `regional` = Regionale Route |
| `ref` | `string` | Routennummer, zum Beispiel R5 oder EV7; fehlt bei Routen ohne Nummer. |
| `signed` | `boolean` | true, wenn die Route beschildert ist. |
| `type` | `string` | Art der Route. Werte: `road` = Radroute auf Straßen und Radwegen; `mtb` = Mountainbike-Strecke |

## Quelle und Aktualität

- Quelle: GIP.at (GIPOGD), Radrouten und Mountainbike-Strecken. Der Stichtag steht in den Kennwerten.

## Hinweise

- `ref` und `difficulty` fehlen bei Routen ohne Nummer bzw. ohne Angabe.

## Weiterverwendung

| | |
|---|---|
| Stil | Gruppe `bike_routes` in `streets`, standardmäßig aus |
| Abfrage | [Tilequery](https://docs.mapx.at/de/api/tilequery.md) mit `bike-routes` als Kachelsatz |

## Quellenangabe

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus dem TileJSON:

```html
mapx.at; Datenquelle: <a href="https://www.gip.gv.at" target="_blank" rel="noopener">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

# GIP-Punkte-Tileset (Park & Ride, Bahnübergänge)

Source: https://docs.mapx.at/de/tilesets/gip-poi/

> Park & Ride, Bike & Ride, Bahnübergänge, Poller, Schranken, Tore und Schutzwege in Österreich aus GIP.at als Vektorkacheln (Tileset gip-poi).

Punkte aus der Graphenintegrationsplattform (GIP.at, GIPOGD): Park & Ride, Bike & Ride, Bahnübergänge, Poller, Schranken und Tore sowie Schutzwege. Die Art steht in `kind`, Park & Ride und Bike & Ride erscheinen ab z12, alle anderen Arten ab z15.

## Kennwerte

**Tileset `gip-poi`**

| Kennwert | Wert |
|---|---|
| Stichtag | `08.07.2026` |
| Format | `mvt` |
| Zoomstufen | `z12–z15` |
| Gebiet (WGS84) | `9,5670 / 46,4991 / 17,0871 / 48,9434 (W/S/O/N)` |
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
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/gip-poi.json" } },
    layers: [{ id: "gip_poi", type: "circle", source: "v", "source-layer": "gip_poi", paint: { "circle-color": "#2d6cdf", "circle-radius": 5, "circle-stroke-color": "#fff", "circle-stroke-width": 1 } }]
  }
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```
TileJSON liefert Zoombereich, Grenzen und Quellenangabe. Den Token bei jeder Anfrage mitschicken: das `transformRequest` aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) in die Kartenoptionen übernehmen.

  

**QGIS**

Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/gip-poi/{z}/{x}/{y}.pbf?access_token=DEIN_TOKEN`, Stil-URL leer lassen.

  

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/gip-poi.json
```

## Ebenen-Referenz

### gip_poi

Geometrie: **Punkt**, Zoom: **z12–15**, Ebene: `gip_poi`. Punktobjekte aus der GIP.at: Park & Ride, Bike & Ride, Bahnübergänge, Poller, Schranken, Tore und Schutzwege.

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Art des Objekts. Werte: `park_and_ride` = Park-&-Ride-Anlage; `park_and_ride_garage` = Park-&-Ride-Garage; `bike_and_ride` = Bike-&-Ride-Anlage; `level_crossing` = Bahnübergang; `bollard` = Poller; `barrier` = Schranke; `gate` = Tor; `crosswalk` = Schutzweg |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt werden soll: 12 für Park & Ride und Bike & Ride, 15 für alle anderen. |
| `name` | `string` | Name der Anlage bei Park & Ride, Bike & Ride und Bahnübergängen (Bahnübergänge: Streckenangabe), sonst nicht vorhanden. |

## Quelle und Aktualität

- Quelle: GIP.at (GIPOGD), Referenzobjekte, Bahnübergänge, Knoten und Querungen. Der Stichtag steht in den Kennwerten.

## Hinweise

- Kapazitäten (Stellplatzzahlen) führen die offenen Daten nicht.
- Park-&-Ride- und Bike-&-Ride-Objekte ohne Lage am Wegenetz fehlen.

## Weiterverwendung

| | |
|---|---|
| Stil | Gruppe `gip_poi` in `streets`, standardmäßig aus |
| Abfrage | [Tilequery](https://docs.mapx.at/de/api/tilequery.md) mit `gip-poi` als Tileset |

## Quellenangabe

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus dem TileJSON:

```html
mapx.at | Datenquelle: <a href="https://www.gip.gv.at" target="_blank" rel="noopener">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/deed.de" target="_blank" rel="noopener">CC BY 4.0</a>
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

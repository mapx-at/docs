# Vektor-Basiskarte (Tileset streets)

Source: https://docs.mapx.at/de/tilesets/streets/

> Vektor-Basiskarte Österreich aus offenen Behördendaten: Nutzung, Gebäude, Straßen, Bahn, Gewässer, Grenzen, Namen, Hausnummern. z0 bis z16.

Die Basiskarte vereint die offenen Daten von BEV, GIP, Umweltbundesamt und BMLUK in 18 Layern. Sie enthält keine OpenStreetMap-Daten. Kataster, Höhenlinien und Relief sind eigene Produkte und lassen sich darüberlegen.

**Nutzungen**

- **Eigener Kartenstil**: Die Layer nach Wunsch stylen (https://docs.mapx.at/de/guides/style-the-basemap/)
- **Web-Karten und Apps**: Vektorkarte mit eigener Gestaltung bis z16
- **Planungs- und Navigationshintergrund**: Straßen, Bahn, Gewässer, Grenzen
- **Hausnummern und Namen**: Adressen, Orte, Berge und Gewässernamen
- **Überlagerung**: Kataster, Höhenlinien und Relief darüber (https://docs.mapx.at/de/guides/overlay-contours/)

## Kennwerte

**Tileset `mapx-basis`**

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
| `aerialway_l` | Linie | z12–16 | `kind` string, `min_zoom` number, `name` string | Seilbahnen und Lifte (DLM): cable_car, gondola, chair_lift, drag_lift … |
| `airport_p` | Punkt | z8–16 | `icao` string, `kind` string, `min_zoom` number, `name` string | Flugplätze (BEV DLM Verkehr): airport, airfield, heliport, military |
| `boundary_l` | Linie | z0–16 | `admin_level` number, `kind` string, `left` string, `min_zoom` number, `right` string | Grenzen (BEV VGD): country, state, district, municipality; left/right Gebiet links/rechts |
| `boundary_label_p` | Punkt | z4–16 | `admin_level` number, `code` string, `kind` string, `min_zoom` number, `name` string | Namen von Bundesland, Bezirk, Gemeinde; code Kennzahl |
| `building_a` | Fläche | z12–16 | `height` number, `kind` string, `max_height` number, `min_zoom` number | Gebäude (DKM) mit Höhe aus dem DLM; kind building oder church, school, hospital, station, castle, hut …; z12–14 zusammengefasst |
| `housenumber_p` | Punkt | z16–16 | `housenumber` string, `kind` string, `street` string | Hausnummern (BEV Adressregister): kind building = Gebäudekoordinate, parcel = Grundstücksadresse |
| `landuse_a` | Fläche | z2–16 | `kind` string, `kind_detail` string, `min_zoom` number, `sort_rank` number | Bodennutzung aus der DKM: residential, garden, farmland, vineyard, alpine_meadow, forest, glacier, rock, water, wetland, transport, quarry, cemetery, recreation |
| `natural_p` | Punkt | z8–16 | `ele` number, `kind` string, `kind_detail` string, `min_zoom` number, `name` string, `sort_rank` number | Berge, Sättel, Täler, Flurnamen, Burgen, Ruinen … |
| `place_p` | Punkt | z0–16 | `capital` string, `kind` string, `min_zoom` number, `name` string, `name_alt` string, `sort_rank` number | Orte (BEV Geographische Namen): city, town, village, hamlet, suburb, locality; capital |
| `poi_p` | Punkt | z13–16 | `ele` number, `kind` string, `kind_detail` string, `min_zoom` number, `name` string, `sort_rank` number | Kirchen, Gipfelkreuze und Einrichtungen (BEV DLM, in Wien zusätzlich Wien-OGD): Schulen, Spitäler, Hütten, Tankstellen, Museen … |
| `rail_l` | Linie | z6–16 | `construction` boolean, `kind` string, `kind_detail` string, `min_zoom` number, `structure` string | Bahn (GIP): rail, narrow_gauge, rack, subway, tram, funicular, ferry |
| `road_l` | Linie | z6–16 | `kind` string, `kind_detail` string, `min_zoom` number, `name` string, `oneway` boolean, `ref` string, `sort_rank` number, `structure` string | Straßen und Wege (GIP): highway, major_road, minor_road, service, track, path; structure bridge/tunnel |
| `ski_a` | Fläche | z10–16 | `kind` string, `min_zoom` number | Skigebiete (Umweltbundesamt, Stand 2017) |
| `transit_p` | Punkt | z10–16 | `kind` string, `min_zoom` number, `name` string | Bahnhöfe, Halte, Liftstationen |
| `water_a` | Fläche | z6–16 | `area_km2` number, `kind` string, `min_zoom` number, `name` string, `sort_rank` number | Stehgewässer (GGN): lake, reservoir, pond |
| `water_l` | Linie | z6–16 | `catchment_km2` number, `kind` string, `min_zoom` number, `name` string, `sort_rank` number, `structure` string | Fließgewässer (GGN): river, stream, canal, ditch |
| `water_label_l` | Linie | z10–16 | `kind` string, `min_zoom` number, `name` string, `sort_rank` number | Gewässernamen entlang der Linie |
| `water_label_p` | Punkt | z6–16 | `kind` string, `min_zoom` number, `name` string, `sort_rank` number | Seenamen, ein Punkt je See |

Alle Felder und Werte jedes Layers: [Basiskarte: Layer-Referenz](https://docs.mapx.at/de/reference/style-spec.md).

## Quelle und Aktualität

| Thema | Quelle | Stand |
|---|---|---|
| Nutzung, Gebäude | BEV Digitale Katastralmappe (DKM), Höhen aus dem DLM | 01.04.2026 |
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
Datenquellen: BEV (DKM, Verwaltungsgrenzen, Adressregister, Geographische Namen), GIP.at, BMLUK (GGN), Umweltbundesamt; CC BY 4.0
```

Mehr: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).

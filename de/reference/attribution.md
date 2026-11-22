# Quellenangabe und Lizenz (CC BY 4.0)

Source: https://docs.mapx.at/de/reference/attribution/

> Welche Quelle bei mapx-Karten und -Daten unter CC BY 4.0 zu nennen ist: fertiger Text je Tileset und API, für MapLibre, Druck und Bilder.

Die offenen Verwaltungsdaten in mapx stehen unter [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Ausnahme sind die Sentinel-2-Kacheln: Sie enthalten veränderte Copernicus-Sentinel-Daten und folgen den Copernicus-Bedingungen, die Quellenangabe steht im TileJSON. Die kommerzielle Nutzung ist erlaubt. Die Quelle und die Lizenz müssen genannt werden, Änderungen dürfen nicht als amtlich dargestellt werden. **Änderungshinweis (CC BY 4.0, Abschnitt 3(a)(1)(B)):** mapx hat die Daten verarbeitet und verändert (umgerechnet, zusammengeführt, generalisiert, interpoliert, eingefärbt, zu Kacheln und Routinggraphen aufbereitet); die Ergebnisse sind keine Originaldaten der Datengeber. Die Quellenangabe muss für die Nutzer auf der Karte selbst erreichbar sein, zum Beispiel als Text in der Ecke oder hinter einem Info-Knopf „i“ auf der Karte, nicht nur im Impressum.

## mapx.at an erster Stelle

Jede Quellenangabe von mapx beginnt mit `mapx.at` (verlinkt auf https://mapx.at), dann folgt ein senkrechter Strich `|` und danach die Datenquellen, untereinander getrennt durch `; `: `mapx.at | <Quelle> (<Lizenz>); …`. Der Strich trennt den Anbieter sichtbar von den Datenquellen und entspricht dem Trenner, den MapLibre zwischen Quellen setzt; der Strichpunkt zwischen den Quellen bleibt, weil Quellnamen Kommas enthalten. Die Stile unter `/v1/styles` bringen `mapx.at` mit Link auf https://mapx.at als ersten Eintrag mit (eigene Quelle `mapx` mit der Ebene `mapx-attribution`); bitte nicht entfernen. Wer Kachelsätze direkt einbindet, bekommt den Link über das TileJSON (`attribution` beginnt mit `<a href="https://mapx.at" …>mapx.at</a> | `), die JSON-Antworten der API über ihr Feld `attribution` (`mapx.at | …`). MapLibre ordnet die Einträge nach Länge, den kürzesten zuerst: eigene Quellen mit sehr kurzer Quellenangabe (unter 68 Zeichen) stehen davor. Bei Leaflet, OpenLayers und statischen Kartenbildern `mapx.at` selbst voranstellen, wie in den Beispielen.

## In einer MapLibre-Karte

MapLibre zeigt die Quellenangabe automatisch an, wenn ein Stil oder ein TileJSON geladen wird (unten rechts). Die Beispiele dieser Doku klappen sie ein, damit sie die Karte nicht verdeckt: `attributionControl: false` in den Optionen der Karte und `map.addControl(new maplibregl.AttributionControl({ compact: true }))` zeigen den Knopf „i“, ein Klick darauf zeigt die Quellen. MapLibre lässt die eingeklappte Form zunächst offen und klappt sie erst bei der ersten Kartenbewegung zu; die Zeile `map.once("load", …)` aus den Beispielen klappt sie gleich nach dem Laden zu. `attributionControl: false` ohne eigene `AttributionControl` blendet die Quellen ganz aus, das ist nicht zulässig. Bei `tiles: [...]` ohne TileJSON den Text in der Quelle selbst ergänzen:

```js
{ type: "raster", tiles: ["https://tiles.mapx.at/v1/tiles/orthophoto/{z}/{x}/{y}.webp?access_token=DEIN_TOKEN"], tileSize: 512,
  maxzoom: 18, attribution: "mapx.at | Orthofoto: BEV, CC BY 4.0" }
```

## In Druck, Berichten und statischen Kartenbildern

Eine sichtbare Zeile neben das Bild setzen, in lesbarer Größe, zum Beispiel `Quelle: mapx.at | BEV – ALS-DTM 1 m (2025-09-15), CC BY 4.0`. Bei mehreren Quellen alle nennen, getrennt durch Strichpunkte.

## Text je Tileset

| Tileset | Text |
|---|---|
| `streets` | mapx.at \| BEV (cadastre, administrative boundaries, address register, geographic names), GIP.at, BMLUK (GGN), Umweltbundesamt, City of Vienna (data.wien.gv.at), Statistik Austria (CC BY 4.0); Datenquelle: [basemap.at](https://basemap.at) (CC BY 4.0); © Österreichisches Adressregister, Stichtagsdaten vom 01.10.2026 |
| `contours` | mapx.at \| Höhenlinien: mapx.at, Datenquelle BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `terrain-rgb` | mapx.at \| Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `dsm` | mapx.at \| Relief: BEV – ALS-DOM 1 m (Stichtag 15.09.2025), CC BY 4.0 |
| `orthophoto` | mapx.at \| Orthophoto: BEV (CC BY 4.0) |
| `orthophoto-flight-blocks` | Das TileJSON nennt nur `mapx.at`; `mapx.at \| BEV, CC BY 4.0` angeben, da die Flugblöcke Metadaten zum Orthofoto des BEV sind. |
| `bike-routes` | mapx.at \| Datenquelle: GIP.at, CC BY 4.0 |
| `gip-poi` | mapx.at \| Datenquelle: GIP.at, CC BY 4.0 |
| `rest-areas` | mapx.at \| ASFINAG, CC BY 4.0 |
| `sharing` | mapx.at \| ÖBB-Personenverkehr AG – Shared-Mobility Standorte, CC BY 4.0 |
| `sentinel2`, `sentinel2-date`, `sentinel2-ndvi`, `sentinel2-ndwi`, `sentinel2-ndsi` | mapx.at \| Contains modified Copernicus Sentinel data 2026 (Jahr der Aufnahmen; steht im TileJSON) |

## Text je API

JSON-Antworten mit Daten tragen ein `attribution`-Feld auf oberster Ebene, es beginnt immer mit `mapx.at | ` vor der Quelle aus der Tabelle. Es ist anzuzeigen, wenn die Daten dargestellt oder weitergegeben werden.

Format: `mapx.at | <Quelle> (<Lizenz>); …`, englisch und ohne Vorsätze wie „Datenquelle:“; für jede `Accept-Language` derselbe Text. Ausnahme sind Wortlaute, die ein Datengeber vorschreibt: Sie stehen unverändert hinter dem Strich (Mobilitätsverbünde, basemap.at, Vermerk des BEV-Adressregisters, siehe unten).

| API | Quelle |
|---|---|
| Geocoding, Search Box, Address | BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0); © Österreichisches Adressregister, Stichtagsdaten vom 01.10.2026 |
| Directions, Isochrone, Matrix, Matching | GIP.at (CC BY 4.0) |
| Maut, Verkehr | ASFINAG / EVIS.AT (CC BY 4.0); GIP.at (CC BY 4.0) (Maut: ASFINAG (CC BY 4.0)) |
| ÖV | ÖBB-Personenverkehr AG (CC BY 4.0); Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at; GIP.at (CC BY 4.0) |
| Wetter | GeoSphere Austria (CC BY 4.0) |
| Klima | GeoSphere Austria APOLIS, SPARTACUS, SNOWGRID-CL, WINFORE (CC BY 4.0) |
| Luftqualität | GeoSphere Austria (CC BY 4.0) |
| Tilequery | Quelle des abgefragten Tilesets, siehe [Tilesets](https://docs.mapx.at/de/tilesets/index.md) |
| Höhe | BEV ALS-DTM 1 m (CC BY 4.0); BEV ALS-DOM 1 m (CC BY 4.0) |

Die Quellenangabe steht auch im TileJSON.

## Vorgeschriebene Wortlaute

- **Mobilitätsverbünde Österreich (Fahrpläne der Verkehrsverbünde):** „Datenquelle: Mobilitätsverbünde Österreich OG – data.mobilitaetsverbuende.at“, wörtlich. Lizenz: [Datenlizenz Mobilitätsverbünde Österreich](https://data.mobilitaetsverbuende.at). mapx hat die Fahrplandaten verändert (zusammengeführt, mit dem ÖBB-Fahrplan und dem Wegenetz GIP.at verknüpft, für die Verbindungssuche aufbereitet). Die Mobilitätsverbünde Österreich OG übernehmen keine Haftung für die Richtigkeit, Vollständigkeit und Aktualität der Daten und für Folgen ihrer Nutzung; es gilt der Haftungsausschluss der Datenlizenz.
- **basemap.at (Auswahl von Gipfel- und Ortsnamen):** „Datenquelle: basemap.at“ mit Link auf https://basemap.at, CC BY 4.0.
- **BEV-Adressregister (Geocoding, Suche, Hausnummern in `streets`):** „© Österreichisches Adressregister, Stichtagsdaten vom TT.MM.JJJJ“, derzeit „© Österreichisches Adressregister, Stichtagsdaten vom 01.10.2026“.
- **Copernicus (Sentinel-2):** „Contains modified Copernicus Sentinel data [Jahr]“ mit dem Jahr der Aufnahmen.
- **Statistik Austria:** Die Einwohnerzahlen (CC BY 4.0) bestimmen die Klassen der Ortsnamen in `streets`.

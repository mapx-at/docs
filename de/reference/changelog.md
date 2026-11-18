# Changelog: Änderungen an API und Tilesets

Source: https://docs.mapx.at/de/reference/changelog/

> Änderungen an den mapx-APIs, Tilesets und dieser Dokumentation, gegliedert in Neu, Geändert, Veraltet und Entfernt.

## 2026-11-18

### Neu
- MCP-Server `https://mcp.mapx.at/mcp` mit 23 Werkzeugen für KI-Assistenten (Claude, ChatGPT, Cursor); eigener Token-Typ KI-Assistent (`mx_mcp_`), der nur am MCP-Server gilt, alternativ Anmeldung über das mapx-Konto (OAuth). Neue Fehler `403` `mcp-token-only` und `mcp-token-required`. Siehe [MCP-Server](https://docs.mapx.at/de/guides/mcp.md).
- [Nutzungsbedingungen](https://mapx.at/nutzungsbedingungen/), verlinkt aus Konto, Registrierung und Doku.

### Geändert
- `tiles.mapx.at`: `GET` auf `.pmtiles`-Dateien nur noch mit genau einer geschlossenen `Range` (`bytes=<start>-<ende>`), sonst `403` [`range-required`](https://docs.mapx.at/de/api/errors.md#range-required). MapLibre mit pmtiles.js ist nicht betroffen. Die Hinweise auf direkte PMTiles-URLs in den Tileset-Seiten und unter Zugang und Limits sind entfernt; Karten laden über Style, TileJSON oder `{z}/{x}/{y}`.
- Suche: `proximity` wirkt jetzt auch bei der Freitextsuche (Umkreis-Abfrage, Wertung aus Treffergüte, Nähe und Bedeutung).
- Einheitliche Fehlerantworten: Auch Anmeldefehler und alle Fehler des Gateways kommen als `application/problem+json` mit eigenem `title` und `detail` je Code (statt `Unauthorized`/`Forbidden` ohne `detail`); `429` hat immer den Titel `Rate limit exceeded`; `405` kommt mit Header `Allow` auch vom Gateway, und zwar vor der Token-Prüfung. Neue Codes `forbidden` (403), `payload-too-large` (413), `uri-too-long` (414) und `upstream-timeout` (504). Siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

### Behoben
- POST-Anfragen mit mehr als 16 KB Körper lieferten `500` statt `413` `payload-too-large`.

### Entfernt
- Das Angebot, PMTiles-Dateien herunterzuladen und selbst zu hosten, entfällt: Anleitung „PMTiles selbst hosten“ (Adresse leitet auf [Karten](https://docs.mapx.at/de/guides/maps.md) um), Dateinamen in den Kennwerten, Hinweise in FAQ, Glossar, Zugang und Limits sowie der Abschnitt „PMTiles direkt“ im [QGIS-Guide](https://docs.mapx.at/de/guides/qgis.md). Karten und Kacheln gibt es über Style, TileJSON und `{z}/{x}/{y}`; die Styles lesen ihre Quellen weiterhin technisch aus PMTiles.
- SDK `mapx-gl` entfällt ersatzlos, ebenso die ausgelieferten Dateien unter `https://map.mapx.at/sdk/v1/` und die SDK-Seiten der Doku (alte Adressen leiten auf die Beispiele um). Stattdessen zeigen die [Beispiele](https://docs.mapx.at/de/examples/index.md) jeden Anwendungsfall mit MapLibre GL JS, OpenLayers oder Leaflet und direkten API-Aufrufen (`fetch`).
- Alte deutsche Namen und doppelte Felder (Übergang seit 2026-11-08) sind entfernt, es gelten nur die englischen Namen. Alte Namen liefern den gleichen Fehler wie jeder unbekannte Name (`404 unknown-style`, `404 unknown-tileset`, `400 invalid-parameter`); die Header `Deprecation`, `Sunset` und `Link: …; rel="successor-version"` entfallen. Zuordnung:

| Bereich | alt → neu |
|---|---|
| Style-IDs | `karte` → `streets`, `karte-morgen` → `streets-dawn`, `karte-abend` → `streets-dusk`, `karte-nacht` → `streets-night`, `karte-verkehr` → `traffic-day`, `luftbild` → `satellite`, `hybrid` → `satellite-streets`, `gelaende` → `outdoors`, `satellit-aktuell` → `satellite-current` |
| Tilesets | `basis`, `mapx-basis` → `streets`; `kataster`, `mapx-kataster` → `cadastre`; `hoehenlinien` → `contours`; `relief`, `relief-terrainrgb` → `terrain-rgb` (Kurzname `dtm` bleibt); `relief-dsm-terrainrgb` → `dsm`; `dop`, `dop-ortho` → `orthophoto`; `dop_flugblock`, `dop-flugblock` → `orthophoto-flight-blocks`; `rastanlagen` → `rest-areas`; `bike_routes` → `bike-routes`; `gip_poi` → `gip-poi`; `sentinel2-datum` → `sentinel2-date` |
| Suchkategorien | `bahnhof` → `train_station`, `haltestelle` → `bus_stop`, `kirche` → `place_of_worship_church`, `religion` → `place_of_worship`, `schule` → `school`, `universitaet` → `university`, `krankenhaus` → `hospital`, `ambulatorium` → `clinic`, `apotheke` → `pharmacy`, `arzt` → `doctor`, `rettung` → `ambulance_station`, `polizei` → `police`, `feuerwehr` → `fire_station`, `gemeindeamt` → `town_hall`, `zoll` → `customs`, `tankstelle` → `gas_station`, `ladestation` → `charging_station`, `parkplatz` → `parking`, `sharing` → `vehicle_sharing`, `rastplatz` → `rest_area`, `faehre` → `ferry_terminal`, `hubschrauberlandeplatz` → `heliport`, `mautstelle` → `toll_booth`, `huette` → `alpine_hut`, `gasthaus` → `restaurant`, `unterkunft` → `lodging`, `campingplatz` → `campground`, `gipfel` → `peak`, `pass` → `mountain_pass`, `hoehle` → `cave`, `schwimmbad` → `swimming_pool`, `sportanlage` → `sports_centre`, `golfplatz` → `golf_course`, `spielplatz` → `playground`, `tiergarten` → `zoo`, `friedhof` → `cemetery`, `bibliothek` → `library`, `theater` → `theatre`, `burg` → `castle`, `denkmal` → `monument`, `sehenswuerdigkeit` → `tourist_attraction`, `markt` → `marketplace`, `forst` → `forest`, `toilette` → `toilets`, `trinkwasser` → `drinking_water` |
| Felder `/v1/parcel` | `gstnr` → `parcel_number`, `kgnr` → `cadastral_municipality_id`, `bauflaeche` → `is_building_land`, `quelle` → `source` |
| Felder Höhe und Tilequery | `stichtag` → `reference_date` |
| Felder `/v1/tiles` und TileJSON | `stand` → `reference_date`, `aliases` entfällt; TileJSON `mapx:stand` → `mapx:reference_date` |
| Feld Adressprüfung | `adrcd` → `address_code` |

## 2026-11-08

### Neu
- API-Tokens: `api.mapx.at`, `tiles.mapx.at`, `routing.mapx.at` und `search.mapx.at` brauchen einen kostenlosen Token von [account.mapx.at](https://account.mapx.at/): öffentlich (`mx_pk_`, an Websites oder eine Android-App gebunden, nur `GET`/`HEAD`) oder geheim (`mx_sk_`). Senden als `Authorization: Bearer` oder `?access_token=`. Neue Fehler `401` (`missing-token`, `invalid-token`, `token-expired`) und `403` (`token-blocked`, `origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `ip-not-allowed`), Grund im Header `X-Mapx-Reason`. Anfragen ohne Token bekommen `401`. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md) und [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).
- Korridor-Graph: `GET /v1/corridor-graph` (`polyline` als polyline6, `buffer` 100 bis 5000 m, `profile` driving, cycling oder walking): kompakter gzip-Teilgraph des Straßennetzes um eine Route (Knoten, gerichtete Arcs mit Fahrzeit und Gewicht, Geometrie, Namen, Straßenklasse, gesetzliche Tempolimits, Abbiegeverbote) für die Neuberechnung ohne Netz.
- Tempolimits: `GET /v1/speedlimits` (`bbox` oder `corridor` als polyline6 plus `buffer`): gesetzliche GIP-Limits als kompaktes GeoJSON.
- Directions: `exclude=ferry` (alle Profile außer `driving-traffic`) und `exclude=toll`; `voice_instructions` und `banner_instructions` auch für `cycling`, `cycling-mtb` und `walking`; `maxspeed` trägt `legal` (`false` kennzeichnet eine modellierte Geschwindigkeit, die kein Limit ist).
- Klima: `GET /v1/climate/{lon},{lat}` und der Abschnitt `climate` im Grundstücks-Dossier: Sonnenpotenzial (kWh/m² und Sonnenstunden je Jahr), Temperatur- und Niederschlagsnormale mit Trend, Hitze-, Frost- und Schneetage, Dürreindex SPEI (GeoSphere Austria, CC BY 4.0).
- Directions: `driving` und `driving-traffic` umfahren verbindliche Sperren (EVIS/ASFINAG) und listen sie in `routes[].closures_avoided`; `exclude` auch für `driving-truck` und `driving-traffic`; `voice_instructions` und `banner_instructions` (Mapbox-Form).
- Header `Server-Timing`, `ETag`, `Last-Modified` (Styles, TileJSON, OpenAPI; bedingte Anfragen antworten mit `304`) und `RateLimit-Policy` je Endpunktgruppe.
- Lkw-Profil `driving-truck` mit `weight`, `height`, `width`, `length` und `truck_type`.
- Fehlerkatalog mit stabilem `code` je Fehler: [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).
- Doku: API-Referenz aus OpenAPI generiert, Playground je API, Guides je Produkt.

### Geändert
- Englische Bezeichner für Styles, Tilesets, Suchkategorien und Antwortfelder.
- Fehler sind `application/problem+json` mit `type`-URI, `code`, `param` und `allowed`; Text englisch, deutsch mit `Accept-Language: de`.
- Die OpenAPI-Beschreibung ist OpenAPI 3.1 mit Antwortschemas und Beispielen.
- Bei Directions ist `overview` standardmäßig `simplified`; `overview=full` liefert jeden Stützpunkt.

### Veraltet (am 2026-11-18 entfernt)
- Style-IDs `karte`, `luftbild`, `gelaende`, `hybrid` und die übrigen deutschen IDs.
- Tileset-Namen `basis`, `kataster`, `hoehenlinien`, `relief`, `dop`, `dop_flugblock`.
- Die 49 deutschen Suchkategorie-IDs (zum Beispiel `bahnhof`, `apotheke`).
- Grundstücksfelder `gstnr`, `kgnr`, `bauflaeche`, `quelle` und Höhenfeld `stichtag`.

### Entfernt
- Die Alias-Pfade `/v1/{tileset}.json` und `/v1/{tileset}/{z}/{x}/{y}.{ext}`. Stattdessen `/v1/tiles/…` verwenden.

## 2026-11-07

### Neu
- Öffentlicher Verkehr: Verbindungen (`/v1/directions/transit`), Abfahrten (`/v1/departures`) und Haltestellen (`/v1/stops`).
- Maut in Directions: `exclude=toll`, `routes[].toll`, `toll_road`, `toll_road_m`.
- Verkehr: `/v1/traffic/*` (Störungen, Tempolimits, Zeichen, Tiles), `incidents` in Directions, Profil `driving-traffic`.
- Tilesets `rastanlagen` (Rastanlagen) und `sharing` (Sharing-Mobilität), Suchkategorien für beide.
- Wetter: Beobachtungen, stündliche Vorhersage und Wetter-Tiles (GeoSphere Austria, CC BY 4.0).
- Static Images: Luftbilder, Gelände- und Kartenstyles als PNG oder WebP mit Overlays.
- Tilequery, Parcel, Search Box, Address Autofill und Address Validation, Batch-Geocoding.
- Directions: `alternatives`, `annotations`, `radiuses`; `matrix`, `isochrone`, `matching`, optimierte Touren und Tourenplanung.
- SDK mapx-gl.

### Geändert
- Reverse Geocoding antwortet in Millisekunden statt in etwa einer Sekunde.
- XYZ-Tiles wurden unter `/v1/tiles` verschoben.
- Attribution in den Antworten; `operationId` an jedem Endpunkt in der OpenAPI.

## Früher

### Neu
- Tilesets DTM, DSM, Höhenlinien, Basiskarte, Kataster, Orthophoto und Flugblöcke als PMTiles, TileJSON und XYZ.
- Höhen- und Profil-API.
- Dokumentation auf Deutsch und Englisch.

## Tileset-Stände

| Tileset | Stichtag | Datei |
|---|---|---|
| terrain-rgb | 20250915 | `relief-terrainrgb-20250915-ed2e4341.pmtiles` |
| dsm | 20250915 | `relief-dsm-terrainrgb-20250915-d19cb434.pmtiles` |

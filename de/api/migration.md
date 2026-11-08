# Migration auf englische Namen


Seit dem 08.11.2026 verwendet die API durchgehend englische Kennungen für Stile, Kachelsätze, Suchkategorien und Antwortfelder. Die alten deutschen Namen funktionieren unverändert weiter, **bis zum 31.05.2027** (Sunset). Danach liefern sie `404` bzw. enthalten die alten Felder nicht mehr.

> Wer bis zum 31.05.2027 nichts ändert, bekommt ab dann Fehler. Die neuen Namen funktionieren schon jetzt; ein Umstieg ist ein reiner Namenstausch, die Daten sind identisch.


## Header bei alten Namen

Eine Anfrage mit altem Namen liefert dieselbe Antwort wie mit dem neuen Namen, dazu drei Header:

| Header | Bedeutung |
|---|---|
| `Deprecation: @1794096000` | Der Name ist seit dem 08.11.2026 abgekündigt ([RFC 9745](https://www.rfc-editor.org/rfc/rfc9745), Zeitstempel in Unix-Sekunden). |
| `Sunset: Mon, 31 May 2027 00:00:00 GMT` | Ab diesem Zeitpunkt entfällt der alte Name ([RFC 8594](https://www.rfc-editor.org/rfc/rfc8594)). |
| `Link: <https://api.mapx.at/v1/styles/streets>; rel="successor-version"` | Die gleiche Ressource unter dem neuen Namen. |

```bash
curl -sI https://api.mapx.at/v1/styles/karte | grep -iE '^(deprecation|sunset|link)'
```

Die Header stehen bei Stilen (`/v1/styles/{id}` samt Sprites und Static Tiles), Static Images (`/v1/static/{style}/…`), Kacheln, TileJSON, Tilequery und bei der Search Box mit alten Kategorie-IDs. In den Antworten mit den alten Feldnamen (Grundstück, Höhe) stehen beide Namen nebeneinander; die alten sind in der [OpenAPI](https://api.mapx.at/v1/openapi.json) als deprecated markiert.

## Stil-IDs

Gilt für `/v1/styles`, `/v1/styles/{id}`, `/v1/styles/{id}/tiles/…`, `/v1/styles/{id}/sprite` und `/v1/static/{style}/…`.

| alt | neu |
|---|---|
| `karte` | `streets` |
| `karte-morgen` | `streets-dawn` |
| `karte-abend` | `streets-dusk` |
| `karte-nacht` | `streets-night` |
| `karte-verkehr` | `traffic-day` |
| `luftbild` | `satellite` |
| `hybrid` | `satellite-streets` |
| `gelaende` | `outdoors` |

Die Metadaten der Stile heißen jetzt `mapx:style` und `mapx:api_style` (früher `mapx:stil`, `mapx:api-stil`). `satellit-aktuell` bleibt unverändert.

## Kachelsätze

Gilt für `/v1/tiles`, TileJSON, XYZ-Kachelpfade und `/v1/tilequery`. `dtm` und `dsm` bleiben. Die Dateinamen der PMTiles ändern sich nicht.

| alt | neu |
|---|---|
| `basis` | `streets` |
| `kataster` | `cadastre` |
| `hoehenlinien` | `contours` |
| `relief` | `terrain-rgb` |
| `dop` | `orthophoto` |
| `dop_flugblock` | `orthophoto-flight-blocks` |

Die Namen in `aliases` der Antwort von `GET /v1/tiles` führen alle gleichwertigen Namen auf. Style-interne Quell-IDs und Ebenen-IDs (zum Beispiel `basis`, `hoehenlinien-index`) bleiben vorerst unverändert.

## Suchkategorien

Gilt für `/v1/searchbox/category/{id}`, `poi_category` und `poi_category_ids`. Der Name (`name`) bleibt über `language=` lokalisiert. Unverändert bleiben: `kindergarten`, `defibrillator`, `park`, `museum`.

| alt | neu |
|---|---|
| `bahnhof` | `train_station` |
| `haltestelle` | `bus_stop` |
| `kirche` | `place_of_worship_church` |
| `religion` | `place_of_worship` |
| `schule` | `school` |
| `universitaet` | `university` |
| `krankenhaus` | `hospital` |
| `ambulatorium` | `clinic` |
| `apotheke` | `pharmacy` |
| `arzt` | `doctor` |
| `rettung` | `ambulance_station` |
| `polizei` | `police` |
| `feuerwehr` | `fire_station` |
| `gemeindeamt` | `town_hall` |
| `zoll` | `customs` |
| `tankstelle` | `gas_station` |
| `ladestation` | `charging_station` |
| `parkplatz` | `parking` |
| `sharing` | `vehicle_sharing` |
| `rastplatz` | `rest_area` |
| `faehre` | `ferry_terminal` |
| `hubschrauberlandeplatz` | `heliport` |
| `mautstelle` | `toll_booth` |
| `huette` | `alpine_hut` |
| `gasthaus` | `restaurant` |
| `unterkunft` | `lodging` |
| `campingplatz` | `campground` |
| `gipfel` | `peak` |
| `pass` | `mountain_pass` |
| `hoehle` | `cave` |
| `schwimmbad` | `swimming_pool` |
| `sportanlage` | `sports_centre` |
| `golfplatz` | `golf_course` |
| `spielplatz` | `playground` |
| `tiergarten` | `zoo` |
| `friedhof` | `cemetery` |
| `bibliothek` | `library` |
| `theater` | `theatre` |
| `burg` | `castle` |
| `denkmal` | `monument` |
| `sehenswuerdigkeit` | `tourist_attraction` |
| `markt` | `marketplace` |
| `forst` | `forest` |
| `toilette` | `toilets` |
| `trinkwasser` | `drinking_water` |

## Felder

| Endpunkt | alt | neu |
|---|---|---|
| `/v1/parcel` | `gstnr` | `parcel_number` |
| `/v1/parcel` | `kgnr` | `cadastral_municipality_id` |
| `/v1/parcel` | `bauflaeche` | `is_building_land` |
| `/v1/parcel` | `quelle` | `source` |
| `/v1/elevation`, `/v1/elevation/profile`, `/v1/elevation/{coordinate}`, `/v1/tilequery` | `stichtag` | `reference_date` |

Bis zum Sunset enthalten die Antworten beide Felder. `/v1/weather/*` liefert die Windrichtung bereits als `wind_direction_deg`.


# Migration to English names

Former German API IDs and field names, their English successors, the deprecation headers and the sunset date 2027-05-31.

Since 2026-11-08 the API uses English identifiers for styles, tilesets, search categories and response fields. The former German names keep working unchanged **until 2027-05-31** (sunset). After that they return `404`, or the old fields are no longer included.

> If nothing changes by 2027-05-31, requests with the old names will fail from then on. The new names already work today; switching is a pure rename, the data is identical.

## Headers on old names

A request using an old name returns the same response as with the new name, plus three headers:

| Header | Meaning |
|---|---|
| `Deprecation: @1794096000` | The name has been deprecated since 2026-11-08 ([RFC 9745](https://www.rfc-editor.org/rfc/rfc9745), Unix timestamp in seconds). |
| `Sunset: Mon, 31 May 2027 00:00:00 GMT` | From this moment the old name is removed ([RFC 8594](https://www.rfc-editor.org/rfc/rfc8594)). |
| `Link: <https://api.mapx.at/v1/styles/streets>; rel="successor-version"` | The same resource under the new name. |

```bash
curl -sI -H "Authorization: Bearer YOUR_TOKEN" https://api.mapx.at/v1/styles/karte | grep -iE '^(deprecation|sunset|link)'
```

The headers appear on styles (`/v1/styles/{id}` including sprites and static tiles), static images (`/v1/static/{style}/…`), tiles, TileJSON, tilequery, and on the Search Box when old category IDs are used. Responses with old field names (parcel, elevation) contain both names side by side; the old ones are marked deprecated in the [OpenAPI](https://api.mapx.at/v1/openapi.json).

## Style IDs

Applies to `/v1/styles`, `/v1/styles/{id}`, `/v1/styles/{id}/tiles/…`, `/v1/styles/{id}/sprite` and `/v1/static/{style}/…`.

| old | new |
|---|---|
| `karte` | `streets` |
| `karte-morgen` | `streets-dawn` |
| `karte-abend` | `streets-dusk` |
| `karte-nacht` | `streets-night` |
| `karte-verkehr` | `traffic-day` |
| `luftbild` | `satellite` |
| `hybrid` | `satellite-streets` |
| `gelaende` | `outdoors` |

Style metadata is now called `mapx:style` and `mapx:api_style` (formerly `mapx:stil`, `mapx:api-stil`). `satellit-aktuell` is unchanged.

## Tilesets

Applies to `/v1/tiles`, TileJSON, XYZ tile paths and `/v1/tilequery`. `dtm` and `dsm` stay. The PMTiles file names do not change.

| old | new |
|---|---|
| `basis` | `streets` |
| `kataster` | `cadastre` |
| `hoehenlinien` | `contours` |
| `relief` | `terrain-rgb` |
| `dop` | `orthophoto` |
| `dop_flugblock` | `orthophoto-flight-blocks` |

The `aliases` of `GET /v1/tiles` list all equivalent names. Style-internal source IDs and layer IDs (for example `basis`, `hoehenlinien-index`) stay unchanged for now.

## Search categories

Applies to `/v1/searchbox/category/{id}`, `poi_category` and `poi_category_ids`. The `name` stays localised via `language=`. Unchanged: `kindergarten`, `defibrillator`, `park`, `museum`.

| old | new |
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

## Fields

| Endpoint | old | new |
|---|---|---|
| `/v1/parcel` | `gstnr` | `parcel_number` |
| `/v1/parcel` | `kgnr` | `cadastral_municipality_id` |
| `/v1/parcel` | `bauflaeche` | `is_building_land` |
| `/v1/parcel` | `quelle` | `source` |
| `/v1/elevation`, `/v1/elevation/profile`, `/v1/elevation/{coordinate}`, `/v1/tilequery` | `stichtag` | `reference_date` |

Until the sunset, responses contain both fields. `/v1/weather/*` already returns wind direction as `wind_direction_deg`.
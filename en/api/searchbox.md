# Search Box

Suggestions, details and category search (railway station, pharmacy, peak …) modelled on Mapbox Search Box.

Search Box is search with a suggestion list: `suggest` while typing, then `retrieve` for the chosen hit. It also searches by category (railway station, pharmacy, peak …). Same index as [geocoding](/en/api/geocoding/) (addresses, places, streets, parcels), extended with official points of interest from BEV DLM, Vienna OGD and GIP (stations, stops). No OpenStreetMap. No key, CORS open.

## Suggestions

**GET https://api.mapx.at/v1/searchbox/suggest**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Search text." },
  { name: "session_token", type: "string", desc: "Any session identifier (at most 128 characters), no effect; for Mapbox compatibility." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: sort equally good hits by distance, <code>distance</code> in the response." },
  { name: "types", type: "string", desc: "Comma-separated: <code>address</code>, <code>street</code>, <code>locality</code>, <code>place</code>, <code>district</code>, <code>region</code>, <code>postcode</code>, <code>poi</code> (<code>country</code>, <code>neighborhood</code>, <code>category</code> have no effect)." },
  { name: "poi_category", type: "string", desc: "Comma-separated category ids (see below), only places in these categories." },
  { name: "bbox", type: "string", desc: "<code>minLon,minLat,maxLon,maxLat</code>." },
  { name: "limit", type: "integer", default: "5", desc: "1 to 10." },
]} />

The response contains `suggestions`; each has `name`, `mapbox_id`, `feature_type` (`address`, `street`, `place`, `poi` …), `full_address`, `place_formatted`, `context` (country, region, district, place, postcode, locality), for places `poi_category` (display name), `poi_category_ids` and `maki` (icon), and `distance` (metres) with `proximity`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/searchbox/suggest?q=bahnhof%20melk&session_token=abc&proximity=15.33,48.22&limit=3"`} />

## Retrieve

**GET https://api.mapx.at/v1/searchbox/retrieve/{id}**

`id` is the `mapbox_id` of a suggestion. The response is a `FeatureCollection` with one `Feature`: `geometry` (point), `properties` as in the suggestion plus `coordinates` (`longitude`, `latitude`), usually `bbox` and `zoom`. `404` for an unknown id. Parameters `session_token` (no effect) and `proximity` (for `distance`).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/searchbox/retrieve/station-at_43_7226?session_token=abc"`} />

## Places in a category

**GET https://api.mapx.at/v1/searchbox/category/{category}**

<ParamTable params={[
  { name: "category", type: "string", required: true, desc: "Category id, several comma-separated (<code>school,kindergarten</code>)." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: nearest first. Without <code>proximity</code> and <code>bbox</code> the order is by importance." },
  { name: "bbox", type: "string", desc: "<code>minLon,minLat,maxLon,maxLat</code>: this area only." },
  { name: "limit", type: "integer", default: "10", desc: "1 to 25." },
]} />

Response: `FeatureCollection` as for `retrieve`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=15.33,48.22&limit=5"`} />

## Category list

**GET https://api.mapx.at/v1/searchbox/list/category**

Returns `listItems` with `canonical_id`, `name` and `icon` (Maki), `language=de` (default) or `en`. The ids are stable; new ones are only added. Sources: churches, peaks, passes, castles and monuments from the place names, stations and stops from GIP, everything else from the POIs (BEV DLM buildings and transport, plus Vienna OGD in Vienna). Only named places are searchable.

| ID | Formerly | Name |
|---|---|---|
| `train_station` | `bahnhof` | Railway station |
| `bus_stop` | `haltestelle` | Public transport stop |
| `place_of_worship_church` | `kirche` | Church and chapel |
| `place_of_worship` | `religion` | Place of worship |
| `school` | `schule` | School |
| `kindergarten` | – | Kindergarten |
| `university` | `universitaet` | University |
| `hospital` | `krankenhaus` | Hospital |
| `clinic` | `ambulatorium` | Clinic |
| `pharmacy` | `apotheke` | Pharmacy |
| `doctor` | `arzt` | Doctor |
| `ambulance_station` | `rettung` | Ambulance station |
| `defibrillator` | – | Defibrillator |
| `police` | `polizei` | Police |
| `fire_station` | `feuerwehr` | Fire station |
| `town_hall` | `gemeindeamt` | Town hall and district office |
| `customs` | `zoll` | Customs |
| `gas_station` | `tankstelle` | Fuel station |
| `charging_station` | `ladestation` | Charging station |
| `parking` | `parkplatz` | Parking |
| `rest_area` | `rastplatz` | Rest area |
| `ferry_terminal` | `faehre` | Ferry terminal |
| `heliport` | `hubschrauberlandeplatz` | Helipad |
| `toll_booth` | `mautstelle` | Toll booth |
| `alpine_hut` | `huette` | Alpine hut and bivouac |
| `restaurant` | `gasthaus` | Inn |
| `lodging` | `unterkunft` | Lodging |
| `campground` | `campingplatz` | Camp site |
| `peak` | `gipfel` | Summit |
| `mountain_pass` | `pass` | Mountain pass |
| `cave` | `hoehle` | Cave and mine |
| `swimming_pool` | `schwimmbad` | Swimming |
| `sports_centre` | `sportanlage` | Sports facility |
| `golf_course` | `golfplatz` | Golf course |
| `playground` | `spielplatz` | Playground |
| `park` | – | Park |
| `zoo` | `tiergarten` | Zoo |
| `cemetery` | `friedhof` | Cemetery |
| `museum` | – | Museum |
| `library` | `bibliothek` | Library |
| `theatre` | `theater` | Theatre |
| `castle` | `burg` | Castle and ruin |
| `monument` | `denkmal` | Monument |
| `tourist_attraction` | `sehenswuerdigkeit` | Attraction |
| `marketplace` | `markt` | Market |
| `forest` | `forst` | Forestry |
| `toilets` | `toilette` | Toilets |
| `drinking_water` | `trinkwasser` | Drinking water |
| `vehicle_sharing` | `sharing` | Sharing station |

## Errors

`400` for a missing `q`, unknown category or invalid type, `404` for `retrieve` of an unknown id, `429` over the limit, `502` if the search is unreachable. See [Errors and limits](/en/api/fehler/).
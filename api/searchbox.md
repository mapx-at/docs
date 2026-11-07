# Search Box

Vorschläge, Details und Kategoriesuche (Bahnhof, Apotheke, Gipfel …) nach dem Vorbild von Mapbox Search Box.

Search Box ist die Suche mit Vorschlagsliste: zuerst `suggest` beim Tippen, dann `retrieve` für den gewählten Treffer. Dazu die Suche nach Kategorien (Bahnhof, Apotheke, Gipfel …). Gleicher Index wie [Geocoding](/api/geocoding/) (Adressen, Orte, Straßen, Grundstücke), erweitert um amtliche Orte von Interesse aus BEV DLM, Wien OGD und GIP (Bahnhöfe, Haltestellen). Kein OpenStreetMap. Kein Schlüssel, CORS offen.

## Vorschläge

**GET https://api.mapx.at/v1/searchbox/suggest**

<ParamTable params={[
  { name: "q", type: "string", required: true, desc: "Suchtext." },
  { name: "session_token", type: "string", desc: "Beliebige Sitzungskennung (höchstens 128 Zeichen), ohne Wirkung; für Mapbox-Kompatibilität." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: gleichwertige Treffer nach Nähe sortieren, <code>distance</code> in der Antwort." },
  { name: "types", type: "string", desc: "Kommagetrennt: <code>address</code>, <code>street</code>, <code>locality</code>, <code>place</code>, <code>district</code>, <code>region</code>, <code>postcode</code>, <code>poi</code> (<code>country</code>, <code>neighborhood</code>, <code>category</code> haben keine Wirkung)." },
  { name: "poi_category", type: "string", desc: "Kommagetrennte Kategorie-IDs (siehe unten), nur Orte dieser Kategorien." },
  { name: "bbox", type: "string", desc: "<code>minLon,minLat,maxLon,maxLat</code>." },
  { name: "limit", type: "integer", default: "5", desc: "1 bis 10." },
]} />

Die Antwort enthält `suggestions`, je Eintrag `name`, `mapbox_id`, `feature_type` (`address`, `street`, `place`, `poi` …), `full_address`, `place_formatted`, `context` (country, region, district, place, postcode, locality), bei Orten `poi_category` (Anzeigename), `poi_category_ids` und `maki` (Symbol) sowie `distance` (Meter) mit `proximity`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/searchbox/suggest?q=bahnhof%20melk&session_token=abc&proximity=15.33,48.22&limit=3"`} />

## Treffer auflösen

**GET https://api.mapx.at/v1/searchbox/retrieve/{id}**

`id` ist der `mapbox_id` eines Vorschlags. Die Antwort ist eine `FeatureCollection` mit einem `Feature`: `geometry` (Punkt), `properties` wie im Vorschlag, dazu `coordinates` (`longitude`, `latitude`), meist `bbox` und `zoom`. `404` bei unbekannter ID. Parameter `session_token` (ohne Wirkung) und `proximity` (für `distance`).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/searchbox/retrieve/station-at_43_7226?session_token=abc"`} />

## Orte einer Kategorie

**GET https://api.mapx.at/v1/searchbox/category/{kategorie}**

<ParamTable params={[
  { name: "kategorie", type: "string", required: true, desc: "Kategorie-ID, mehrere kommagetrennt (<code>school,kindergarten</code>)." },
  { name: "proximity", type: "string", desc: "<code>lon,lat</code>: nächste zuerst. Ohne <code>proximity</code> und <code>bbox</code> ist die Reihenfolge nach Wichtigkeit." },
  { name: "bbox", type: "string", desc: "<code>minLon,minLat,maxLon,maxLat</code>: nur dieses Gebiet." },
  { name: "limit", type: "integer", default: "10", desc: "1 bis 25." },
]} />

Antwort: `FeatureCollection` wie bei `retrieve`.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=15.33,48.22&limit=5"`} />

## Kategorieliste

**GET https://api.mapx.at/v1/searchbox/list/category**

Liefert `listItems` mit `canonical_id`, `name` und `icon` (Maki), `language=de` (Standard) oder `en`. Die IDs sind stabil, neue kommen nur dazu. Zuordnung der Quellen: Kirchen, Gipfel, Pässe, Schlösser und Denkmäler aus den Geonamen, Bahnhöfe und Haltestellen aus der GIP, alles Übrige aus den POIs (BEV DLM Bauten und Verkehr, in Wien zusätzlich Wien OGD). Nur benannte Orte sind durchsuchbar.

| ID | Früher | Name |
|---|---|---|
| `train_station` | `bahnhof` | Bahnhof |
| `bus_stop` | `haltestelle` | Haltestelle |
| `place_of_worship_church` | `kirche` | Kirche und Kapelle |
| `place_of_worship` | `religion` | Religiöse Stätte |
| `school` | `schule` | Schule |
| `kindergarten` | – | Kindergarten |
| `university` | `universitaet` | Universität und Hochschule |
| `hospital` | `krankenhaus` | Krankenhaus |
| `clinic` | `ambulatorium` | Ambulatorium |
| `pharmacy` | `apotheke` | Apotheke |
| `doctor` | `arzt` | Arzt |
| `ambulance_station` | `rettung` | Rettungsstation |
| `defibrillator` | – | Defibrillator |
| `police` | `polizei` | Polizei |
| `fire_station` | `feuerwehr` | Feuerwehr |
| `town_hall` | `gemeindeamt` | Gemeindeamt und Bezirkshauptmannschaft |
| `customs` | `zoll` | Zoll |
| `gas_station` | `tankstelle` | Tankstelle |
| `charging_station` | `ladestation` | E-Ladestation |
| `parking` | `parkplatz` | Parkplatz |
| `rest_area` | `rastplatz` | Raststation und Rastplatz |
| `ferry_terminal` | `faehre` | Fähre und Schiffsstation |
| `heliport` | `hubschrauberlandeplatz` | Hubschrauberlandeplatz |
| `toll_booth` | `mautstelle` | Mautstelle |
| `alpine_hut` | `huette` | Hütte und Biwak |
| `restaurant` | `gasthaus` | Gasthaus |
| `lodging` | `unterkunft` | Unterkunft |
| `campground` | `campingplatz` | Campingplatz |
| `peak` | `gipfel` | Gipfel |
| `mountain_pass` | `pass` | Pass und Sattel |
| `cave` | `hoehle` | Höhle und Bergwerk |
| `swimming_pool` | `schwimmbad` | Schwimmbad und Badestelle |
| `sports_centre` | `sportanlage` | Sportanlage |
| `golf_course` | `golfplatz` | Golfplatz |
| `playground` | `spielplatz` | Spielplatz |
| `park` | – | Park |
| `zoo` | `tiergarten` | Tierpark |
| `cemetery` | `friedhof` | Friedhof |
| `museum` | – | Museum |
| `library` | `bibliothek` | Bücherei |
| `theatre` | `theater` | Theater |
| `castle` | `burg` | Burg, Schloss und Ruine |
| `monument` | `denkmal` | Denkmal und Gipfelkreuz |
| `tourist_attraction` | `sehenswuerdigkeit` | Sehenswürdigkeit |
| `marketplace` | `markt` | Markt |
| `forest` | `forst` | Forst und Jagdhaus |
| `toilets` | `toilette` | Toilette |
| `drinking_water` | `trinkwasser` | Trinkwasser und Brunnen |
| `vehicle_sharing` | `sharing` | Sharing-Station |

## Fehler

`400` bei fehlendem `q`, unbekannter Kategorie oder ungültigem Typ, `404` bei `retrieve` einer unbekannten ID, `429` über dem Limit, `502` wenn die Suche nicht erreichbar ist. Siehe [Fehler und Limits](/api/fehler/).
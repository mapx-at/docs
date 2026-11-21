# Basiskarte: Layer und Style-Spezifikation

Source: https://docs.mapx.at/de/reference/style-spec/

> Alle Layer des mapx-Vektor-Tilesets streets mit Feldern und Werten sowie die fertigen Style-IDs. Referenz für eigene MapLibre-Styles.

## Style-IDs

| ID | Aussehen |
|---|---|
| `streets`, `streets-dawn`, `streets-dusk`, `streets-night` | Basiskarte in vier Lichtstimmungen |
| `satellite`, `satellite-streets` | Orthophoto, wahlweise mit Straßen und Beschriftung |
| `light`, `dark` | Ruhige Grundkarte für eigene Daten, hell bzw. dunkel |
| `outdoors` | Höhenlinien, Schummerung, Wege |
| `outdoors-winter` | Gelände in Schneefarben, Skigebiete und Lifte hervorgehoben |
| `traffic-day` | Basiskarte mit Verkehrsebene |

Styles sind MapLibre-Style-Dokumente (Version 8) unter `https://api.mapx.at/v1/styles/{id}`. Die Metadaten heißen `mapx:style` und `mapx:api_style`. Siehe [Styles API](https://docs.mapx.at/de/api/styles.md).

## Layer von `streets`

Jeder Layer der [`streets`](https://docs.mapx.at/de/tilesets/streets.md) mit seinen Feldern. Bei Feldern mit geschlossener Liste stehen alle möglichen Werte dabei. Layernamen enden auf `_a` (Fläche), `_l` (Linie) oder `_p` (Punkt).

Gemeinsame Felder: `min_zoom` ist die erste Zoomstufe, ab der ein Objekt gezeigt werden soll (für Filter mit gleichmäßiger Dichte); `sort_rank` ordnet Objekte innerhalb eines Layers (kleiner = wichtiger bzw. zuerst gezeichnet).

## Fläche und Nutzung

### landuse_a

Geometrie: **Fläche**, Zoom: **z2–16**, Layer: `landuse_a`. Flächennutzung und Bodenbedeckung (Flächen).

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Nutzungsart der Fläche. Werte: `residential` = Siedlung; `garden` = Garten, Grünfläche; `farmland` = Acker, Wiese, Weide; `forest` = Wald; `glacier` = Gletscher; `transport` = Verkehrsfläche; `rock` = Fels und Geröll; `water` = Gewässer; `wetland` = Feuchtgebiet; `cemetery` = Friedhof; `recreation` = Freizeitfläche; `vineyard` = Weingarten; `alpine_meadow` = Alm, Alpenwiese; `quarry` = Abbau, Halde, Deponie |
| `kind_detail` | `string` | Genauere Unterteilung; nur bei einem Teil der Nutzungsarten gesetzt. Werte: `orchard` = Obstanlage, Dauerkultur (bei kind `farmland`); `building_area` = Gebäudefläche (bei kind `residential`); `industrial` = Betriebs- und Industriefläche (bei kind `residential`); `building_yard` = Gebäudenebenfläche (Hof, Vorplatz) (bei kind `residential`); `parking` = Parkplatz (bei kind `transport`); `verge` = Verkehrsrandfläche (bei kind `transport`); `rail` = Bahnanlage (bei kind `transport`); `road` = Straßenverkehrsfläche (bei kind `transport`); `krummholz` = Krummholz (Latschen) (bei kind `forest`); `scrub` = Verbuschte Fläche (bei kind `forest`); `forest_road` = Forststraße (bei kind `forest`); `river` = Fließgewässer (bei kind `water`); `lake` = Stehendes Gewässer (bei kind `water`); `bank` = Gewässerrandfläche (bei kind `water`); `bare` = Vegetationsarme Fläche (bei kind `rock`); `scree` = Fels- und Geröllfläche (bei kind `rock`) |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `sort_rank` | `number` | Zeichenreihenfolge der Flächen (1 = unten, 10 = oben). |

### building_a

Geometrie: **Fläche**, Zoom: **z12–16**, Layer: `building_a`. Gebäudegrundrisse.

| Feld | Typ | Beschreibung |
|---|---|---|
| `height` | `number` | Gebäudehöhe in Metern. |
| `kind` | `string` | Gebäudeart; Standard ist ein gewöhnliches Gebäude, markante Gebäude sind gesondert gekennzeichnet. Werte: `building` = Gebäude (allgemein); `church` = Kirche; `chapel` = Kapelle; `castle` = Burg, Schloss; `fire_station` = Feuerwehr; `hospital` = Krankenhaus; `pharmacy` = Apotheke; `police` = Polizei; `power` = Kraftwerk, Umspannwerk; `school` = Schule; `station` = Bahnhofsgebäude; `townhall` = Gemeinde- oder Bezirksamt; `hut` = Schutzhütte |
| `max_height` | `number` | Größte Höhe des Gebäudes in Metern. |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |

### ski_a

Geometrie: **Fläche**, Zoom: **z10–16**, Layer: `ski_a`. Skigebiete (Flächen).

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Art der Fläche. Werte: `ski_area` = Skigebiet |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |

## Verkehr

### road_l

Geometrie: **Linie**, Zoom: **z6–16**, Layer: `road_l`. Straßen und Wege (Linien).

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Straßenklasse. Werte: `highway` = Autobahn, Schnellstraße; `major_road` = Hauptstraße, Landesstraße; `minor_road` = Gemeinde- und Nebenstraße, Fußgängerzone; `service` = Zufahrt, Parkplatz- und Betriebsweg; `track` = Wirtschafts-, Forst- und Feldweg; `path` = Fuß-, Rad- und Wanderweg, Stiege |
| `kind_detail` | `string` | Genauere Straßenart. Werte: `motorway` = Autobahn, Schnellstraße (bei kind `highway`); `motorway_link` = Auf- oder Abfahrt (bei kind `highway`); `motorway_construction` = Autobahn im Bau (bei kind `highway`); `primary` = Hauptstraße (B-Straße) (bei kind `major_road`); `primary_construction` = Hauptstraße im Bau (bei kind `major_road`); `secondary` = Landesstraße (bei kind `major_road`); `minor` = Gemeinde- oder Nebenstraße (bei kind `minor_road`); `pedestrian` = Fußgängerzone (bei kind `minor_road`); `service` = Zufahrt, Betriebsweg (bei kind `service`); `track` = Wirtschafts-, Forst- oder Feldweg (bei kind `track`); `path` = Fußweg, Geh- und Radweg (bei kind `path`); `cycleway` = Radweg (bei kind `path`); `steps` = Stiege, Treppe (bei kind `path`); `trail` = Wanderweg, Steig (bei kind `path`) |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `oneway` | `boolean` | true bei Einbahnstraßen (in Digitalisierungsrichtung). |
| `ref` | `string` | Straßennummer, z. B. A1 oder B3. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |
| `structure` | `string` | Bauwerksart: Brücke oder Tunnel. Fehlt bei Strecken am Boden. Werte: `bridge` = Brücke; `tunnel` = Tunnel |

### rail_l

Geometrie: **Linie**, Zoom: **z6–16**, Layer: `rail_l`. Bahnstrecken und Schifffahrtslinien (Linien).

| Feld | Typ | Beschreibung |
|---|---|---|
| `construction` | `boolean` | true, wenn die Strecke noch im Bau ist. |
| `kind` | `string` | Bahnart. Werte: `rail` = Normalspurbahn; `narrow_gauge` = Schmalspurbahn; `rack` = Zahnradbahn; `subway` = U-Bahn; `tram` = Straßenbahn; `funicular` = Standseilbahn; `ferry` = Fähre, Schifffahrtslinie |
| `kind_detail` | `string` | Rang der Strecke (bei Normalspur-, Schmalspur- und Zahnradbahn). Werte: `main` = Hochrangige Strecke; `branch` = Nebenstrecke; `siding` = Anschluss- oder Nebengleis |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `structure` | `string` | Bauwerksart: Brücke oder Tunnel. Fehlt bei Strecken am Boden. Werte: `bridge` = Brücke; `tunnel` = Tunnel |

### aerialway_l

Geometrie: **Linie**, Zoom: **z12–16**, Layer: `aerialway_l`. Seilbahnen und Lifte (Linien).

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Art der Anlage. Werte: `cable_car` = Pendelseilbahn; `gondola` = Kabinenbahn (Umlaufbahn); `chair_lift` = Sessellift; `drag_lift` = Schlepplift; `goods` = Materialseilbahn (auch Lawinensprengseilbahn); `inclined_lift` = Schrägaufzug; `zip_line` = Seilrutsche |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |

### transit_p

Geometrie: **Punkt**, Zoom: **z10–16**, Layer: `transit_p`. Bahnhöfe, Haltestellen und Seilbahnstationen.

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Art der Station. Werte: `train_station` = Bahnhof; `halt` = Haltestelle; `aerialway_station` = Seilbahnstation |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |

### airport_p

Geometrie: **Punkt**, Zoom: **z8–16**, Layer: `airport_p`. Flughäfen, Flugplätze und Hubschrauberlandeplätze.

| Feld | Typ | Beschreibung |
|---|---|---|
| `iata` | `string` | IATA-Code (3 Buchstaben), falls vorhanden. |
| `icao` | `string` | ICAO-Code (4 Buchstaben), falls vorhanden. |
| `kind` | `string` | Art des Flugplatzes. Werte: `airport` = Verkehrsflughafen; `airfield` = Flugplatz; `heliport` = Hubschrauberlandeplatz; `military` = Militärflugplatz |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |

## Gewässer

### water_a

Geometrie: **Fläche**, Zoom: **z6–16**, Layer: `water_a`. Stehende Gewässer (Flächen).

| Feld | Typ | Beschreibung |
|---|---|---|
| `area_km2` | `number` | Fläche in Quadratkilometern. |
| `kind` | `string` | Art des Gewässers. Werte: `lake` = See; `pond` = Teich, Weiher; `reservoir` = Stausee, Speicher |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |

### water_l

Geometrie: **Linie**, Zoom: **z6–16**, Layer: `water_l`. Fließgewässer (Linien).

| Feld | Typ | Beschreibung |
|---|---|---|
| `catchment_km2` | `number` | Einzugsgebiet in Quadratkilometern. |
| `kind` | `string` | Art des Gewässers. Werte: `river` = Fluss; `stream` = Bach; `canal` = Kanal; `ditch` = Graben |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |
| `structure` | `string` | Bauwerksart: verrohrte oder unterirdische Strecke. Werte: `tunnel` = Tunnel, Verrohrung |

### water_label_p

Geometrie: **Punkt**, Zoom: **z6–16**, Layer: `water_label_p`. Beschriftungspunkte für Seen und Teiche.

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Art des Gewässers. Werte: `lake` = See; `pond` = Teich, Weiher; `reservoir` = Stausee, Speicher |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |

### water_label_l

Geometrie: **Linie**, Zoom: **z10–16**, Layer: `water_label_l`. Beschriftungslinien für Flüsse und Kanäle.

| Feld | Typ | Beschreibung |
|---|---|---|
| `kind` | `string` | Art des Gewässers. Werte: `river` = Fluss; `canal` = Kanal |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |

## Grenzen

### boundary_l

Geometrie: **Linie**, Zoom: **z0–16**, Layer: `boundary_l`. Verwaltungsgrenzen (Linien).

| Feld | Typ | Beschreibung |
|---|---|---|
| `admin_level` | `number` | Verwaltungsebene der Grenze (2 Staat, 4 Bundesland, 6 Bezirk, 8 Gemeinde). Werte: `2` = Staatsgrenze; `4` = Landesgrenze (Bundesland); `6` = Bezirksgrenze; `8` = Gemeindegrenze |
| `kind` | `string` | Art der Grenze. Werte: `country` = Staatsgrenze; `state` = Bundesland; `district` = Bezirk; `municipality` = Gemeinde |
| `left` | `string` | Name der Verwaltungseinheit links der Linie (bei Staatsgrenzen nicht gesetzt). |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `right` | `string` | Name der Verwaltungseinheit rechts der Linie (bei Staatsgrenzen das angrenzende Gebiet). |

### boundary_label_p

Geometrie: **Punkt**, Zoom: **z4–16**, Layer: `boundary_label_p`. Beschriftungspunkte für Bundesländer, Bezirke und Gemeinden.

| Feld | Typ | Beschreibung |
|---|---|---|
| `admin_level` | `number` | Verwaltungsebene. Werte: `4` = Bundesland; `6` = Bezirk; `8` = Gemeinde |
| `code` | `string` | Amtlicher Kennzahl-Code der Einheit. |
| `kind` | `string` | Art der Verwaltungseinheit. Werte: `state` = Bundesland; `district` = Bezirk; `municipality` = Gemeinde |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |

## Namen und Orte

### place_p

Geometrie: **Punkt**, Zoom: **z0–16**, Layer: `place_p`. Orte und Siedlungen.

| Feld | Typ | Beschreibung |
|---|---|---|
| `capital` | `string` | Hauptstadtfunktion des Ortes; nur gesetzt, wenn zutreffend. Werte: `country` = Hauptstadt des Staates; `state` = Landeshauptstadt; `district` = Bezirkshauptort |
| `kind` | `string` | Ortskategorie. Werte: `city` = Stadt; `town` = Marktgemeinde, Kleinstadt; `village` = Dorf; `hamlet` = Weiler, kleines Dorf; `suburb` = Stadtteil; `locality` = Einzelhaus, Gehöft |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `name_alt` | `string` | Alternativer Name, falls vorhanden. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |

### natural_p

Geometrie: **Punkt**, Zoom: **z8–16**, Layer: `natural_p`. Geländenamen, Gipfel, Pässe, Gletscher sowie benannte Burgen, Schlösser und Kleindenkmäler.

| Feld | Typ | Beschreibung |
|---|---|---|
| `ele` | `number` | Seehöhe in Metern. |
| `kind` | `string` | Art des Objekts. Werte: `peak` = Gipfel; `saddle` = Pass, Sattel; `ridge` = Bergteil, Grat, Rücken; `valley` = Tal; `glacier` = Gletscher; `mountain_range` = Gebirge; `area` = Gebiet, Landschaft; `field_name` = Flurname, Riedname; `castle` = Burg; `palace` = Schloss; `ruins` = Ruine; `tower` = Turm, Warte; `minor_monument` = Kleindenkmal (Bildstock, Wegkreuz); `monument` = Denkmal; `other` = Sonstiges benanntes Objekt |
| `kind_detail` | `string` | Zusatzinfo zur Darstellung. Werte: `summit_label` = Als Gipfel dargestellter Bergteil, Pass oder Tal; `sonstiges` = Sonstiges Objekt |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |

### poi_p

Geometrie: **Punkt**, Zoom: **z13–16**, Layer: `poi_p`. Einrichtungen und Orte von Interesse (Punkte).

| Feld | Typ | Beschreibung |
|---|---|---|
| `ele` | `number` | Seehöhe in Metern. |
| `kind` | `string` | Art der Einrichtung. Werte: `bivouac` = Biwakschachtel; `alpine_hut` = Schutzhütte; `inn` = Ausflugsgasthof; `mine` = Bergwerk; `cave` = Höhle; `camp_site` = Campingplatz; `pharmacy` = Apotheke; `police` = Polizei; `fire_station` = Feuerwehr; `hospital` = Krankenhaus; `clinic` = Ambulatorium, Klinik; `ambulance_station` = Rettungsstelle; `school` = Schule; `music_school` = Musikschule; `kindergarten` = Kindergarten; `university` = Universität; `survey_office` = Vermessungsamt; `district_office` = Bezirkshauptmannschaft, Magistrat; `forestry` = Forstbetrieb; `townhall` = Gemeindeamt; `customs` = Zoll; `swimming_pool` = Schwimmbad; `swimming_area` = Strandbad, Badestelle; `golf_course` = Golfplatz; `pitch` = Sportplatz; `stadium` = Stadion; `sports_centre` = Sportanlage; `park` = Park; `zoo` = Tierpark; `open_air_museum` = Freilichtmuseum; `archaeological_site` = Archäologische Ausgrabung; `theatre` = Freilichtbühne; `attraction` = Sehenswürdigkeit, Ausstellungsgelände; `cemetery` = Friedhof; `visitor_centre` = Besucherzentrum; `ferry_terminal` = Fähr- oder Schiffsstation; `fuel` = Tankstelle; `parking` = Parkplatz, Parkgarage; `rest_area` = Rastplatz, Raststation; `toll_booth` = Mautstelle; `helipad` = Hubschrauberlandeplatz; `church` = Kirche; `chapel` = Kapelle; `monastery` = Kloster, Stift; `summit_cross` = Gipfelkreuz; `place_of_worship` = Sonstiger Kultort; `museum` = Museum; `library` = Bücherei; `drinking_water` = Trinkwasserstelle; `toilets` = öffentliche Toilette; `playground` = Spielplatz; `charging_station` = Ladestation für Elektrofahrzeuge; `defibrillator` = Defibrillator; `doctor` = Arzt; `lodging` = Unterkunft; `viewpoint` = Aussichtspunkt; `market` = Markt |
| `kind_detail` | `string` | Genauere Unterteilung; nur bei einem Teil der Einrichtungsarten gesetzt. Werte: `public` = öffentliche Apotheke (bei kind `pharmacy`); `branch` = Filialapotheke (bei kind `pharmacy`); `hospital` = Krankenhausapotheke (bei kind `pharmacy`); `inspection` = Polizeiinspektion (bei kind `police`); `motorway` = Autobahnpolizei (bei kind `police`); `border` = Grenzpolizei (bei kind `police`); `traffic` = Verkehrsinspektion (bei kind `police`); `district_command` = Bezirkspolizeikommando (bei kind `police`); `city_command` = Stadtpolizeikommando (bei kind `police`); `state_directorate` = Landespolizeidirektion (bei kind `police`); `volunteer` = Freiwillige Feuerwehr (bei kind `fire_station`); `professional` = Berufsfeuerwehr (bei kind `fire_station`); `company` = Betriebsfeuerwehr (bei kind `fire_station`); `district_command` = Bezirksfeuerwehrkommando (bei kind `fire_station`); `state_association` = Landesfeuerwehrverband (bei kind `fire_station`); `acute` = Akutkrankenhaus (bei kind `hospital`); `other` = Sonstiges Krankenhaus (bei kind `hospital`); `outpatient` = Ambulatorium (bei kind `clinic`); `primary` = Volksschule (bei kind `school`); `secondary` = Mittelschule (bei kind `school`); `academic_secondary` = Allgemein bildende höhere Schule (AHS) (bei kind `school`); `general` = Sonstige allgemein bildende Schule (bei kind `school`); `prevocational` = Polytechnische Schule (bei kind `school`); `special` = Sonderschule (bei kind `school`); `vocational` = Berufsschule (bei kind `school`); `commercial` = Kaufmännische Schule (bei kind `school`); `technical` = Technische oder gewerbliche Schule (bei kind `school`); `hospitality` = Wirtschaftsberufliche Schule (Tourismus, Wirtschaft) (bei kind `school`); `social` = Sozialberufliche Schule (bei kind `school`); `health` = Gesundheits- und Krankenpflegeschule (bei kind `school`); `agricultural` = Land- und forstwirtschaftliche Schule (bei kind `school`); `teacher_training` = Pädagogische Schule, Lehrerbildung (bei kind `school`); `other` = Sonstige Bildungseinrichtung (bei kind `school`); `district` = Bezirkshauptmannschaft (bei kind `district_office`); `city` = Magistrat (bei kind `district_office`); `forest_house` = Forsthaus (bei kind `forestry`); `office` = Forstverwaltung (bei kind `forestry`); `hunting_lodge` = Jagdhütte (bei kind `forestry`); `indoor` = Hallenbad (bei kind `swimming_pool`); `indoor_outdoor` = Hallen- und Freibad (bei kind `swimming_pool`); `outdoor` = Freibad (bei kind `swimming_pool`); `beach` = Strandbad (bei kind `swimming_area`); `tennis` = Tennisplatz (bei kind `pitch`); `soccer` = Fußballplatz (bei kind `pitch`); `ball_sports` = Ballsportanlage (bei kind `sports_centre`); `multi_sport` = Mehrzwecksportanlage (bei kind `sports_centre`); `motor_sport` = Motorsportanlage (bei kind `sports_centre`); `winter_sport` = Wintersportanlage (bei kind `sports_centre`); `exhibition_ground` = Ausstellungsgelände (bei kind `attraction`); `zoo` = Tierpark (bei kind `zoo`); `wildlife_park` = Wildpark (bei kind `zoo`); `open_air` = Freilichtbühne (bei kind `theatre`); `forest` = Waldfriedhof (bei kind `cemetery`); `historic` = Historischer Friedhof (bei kind `cemetery`); `pet` = Tierfriedhof (bei kind `cemetery`); `national_park` = Nationalpark-Besucherzentrum (bei kind `visitor_centre`); `ferry` = Fähre (bei kind `ferry_terminal`); `ship_station` = Schiffsstation (bei kind `ferry_terminal`); `picnic` = Rastplatz (bei kind `rest_area`); `service_area` = Raststation (bei kind `rest_area`); `eintuermig` = Kirche mit einem Turm (bei kind `church`); `mehrtuermig` = Kirche mit mehreren Türmen (bei kind `church`); `eintuermig` = Kloster mit einem Turm (bei kind `monastery`); `mehrtuermig` = Kloster mit mehreren Türmen (bei kind `monastery`); `fountain` = Trinkbrunnen (bei kind `drinking_water`); `hydrant` = Trinkhydrant (bei kind `drinking_water`); `garage` = Parkgarage (bei kind `parking`); `park_and_ride` = Park-and-Ride (bei kind `parking`); `synagogue` = Synagoge (bei kind `place_of_worship`); `christian` = Christliche Gemeinschaft (bei kind `place_of_worship`); `muslim` = Islamische Gemeinschaft (bei kind `place_of_worship`); `buddhist` = Buddhistische Gemeinschaft (bei kind `place_of_worship`); `other` = Sonstige Gemeinschaft (bei kind `place_of_worship`); `hotel` = Hotel (bei kind `lodging`); `pension` = Pension (bei kind `lodging`) |
| `min_zoom` | `number` | Zoomstufe, ab der das Objekt dargestellt wird. |
| `name` | `string` | Name des Objekts. |
| `sort_rank` | `number` | Darstellungsrang; kleinere Zahl = wichtiger bzw. weiter unten gezeichnet. |

### housenumber_p

Geometrie: **Punkt**, Zoom: **z16–16**, Layer: `housenumber_p`. Hausnummern.

| Feld | Typ | Beschreibung |
|---|---|---|
| `housenumber` | `string` | Hausnummer. |
| `kind` | `string` | Bezugsobjekt der Hausnummer. Werte: `building` = Gebäude; `parcel` = Grundstück |
| `street` | `string` | Straßenname. |

# Basemap layers and style specification

Source: https://docs.mapx.at/reference/style-spec/

> All layers of the mapx streets vector tileset with fields and values, plus the ready-made style IDs. Reference for custom MapLibre styles.

## Style IDs

| ID | Look |
|---|---|
| `streets`, `streets-dawn`, `streets-dusk`, `streets-night` | Base map in four light moods |
| `satellite`, `satellite-streets` | Orthophoto, optionally with roads and labels |
| `outdoors` | Contours, hillshade, trails |
| `traffic-day` | Base map with the traffic layer |

Styles are MapLibre style documents (version 8) at `https://api.mapx.at/v1/styles/{id}`. Their metadata keys are `mapx:style` and `mapx:api_style`. See the [Styles API](https://docs.mapx.at/api/styles.md).

## Layers of `streets`

Every layer of [`streets`](https://docs.mapx.at/tilesets/streets.md) with its fields. For fields with a closed list, all values are listed. Layer names end in `_a` (polygon), `_l` (line) or `_p` (point).

Common fields: `min_zoom` is the first zoom level at which a feature is intended to be shown (use it in filters for an even density); `sort_rank` orders features within a layer (smaller = more important or drawn first).

## Land use and buildings

### landuse_a

Geometry: **Polygon**, Zoom: **z2–16**, Layer: `landuse_a`. Land use and land cover (polygons).

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Land-use class. Values: `residential` = Settlement; `garden` = Garden, green space; `farmland` = Cropland, meadow, pasture; `forest` = Forest; `glacier` = Glacier; `transport` = Transport area; `rock` = Rock and scree; `water` = Water; `wetland` = Wetland; `cemetery` = Cemetery; `recreation` = Recreation area; `vineyard` = Vineyard; `alpine_meadow` = Alpine pasture; `quarry` = Quarry, spoil heap, landfill |
| `kind_detail` | `string` | Finer subdivision; set for some land-use classes only. Values: `orchard` = Orchard, permanent crop (with kind `farmland`); `building_area` = Building area (with kind `residential`); `industrial` = Commercial and industrial area (with kind `residential`); `building_yard` = Building yard (with kind `residential`); `parking` = Parking (with kind `transport`); `verge` = Road verge (with kind `transport`); `rail` = Railway area (with kind `transport`); `road` = Road area (with kind `transport`); `krummholz` = Dwarf pine (with kind `forest`); `scrub` = Scrubland (with kind `forest`); `forest_road` = Forest road (with kind `forest`); `river` = Flowing water (with kind `water`); `lake` = Standing water (with kind `water`); `bank` = Water bank (with kind `water`); `bare` = Sparsely vegetated area (with kind `rock`); `scree` = Rock and scree (with kind `rock`) |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `sort_rank` | `number` | Drawing order of the areas (1 = bottom, 10 = top). |

### building_a

Geometry: **Polygon**, Zoom: **z12–16**, Layer: `building_a`. Building footprints.

| Field | Type | Description |
|---|---|---|
| `height` | `number` | Building height in metres. |
| `kind` | `string` | Building type; ordinary buildings are plain, notable buildings are marked separately. Values: `building` = Building (general); `church` = Church; `chapel` = Chapel; `castle` = Castle, palace; `fire_station` = Fire station; `hospital` = Hospital; `pharmacy` = Pharmacy; `police` = Police; `power` = Power plant, substation; `school` = School; `station` = Station building; `townhall` = Town hall or district office; `hut` = Mountain hut |
| `max_height` | `number` | Maximum height of the building in metres. |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |

### ski_a

Geometry: **Polygon**, Zoom: **z10–16**, Layer: `ski_a`. Ski areas (polygons).

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Type of area. Values: `ski_area` = Ski area |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |

## Transport

### road_l

Geometry: **Line**, Zoom: **z6–16**, Layer: `road_l`. Roads and paths (lines).

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Road class. Values: `highway` = Motorway, expressway; `major_road` = Main road; `minor_road` = Minor road, pedestrian street; `service` = Service road; `track` = Track; `path` = Path, footway, cycleway, steps |
| `kind_detail` | `string` | Detailed road type. Values: `motorway` = Motorway, expressway (with kind `highway`); `motorway_link` = Slip road (with kind `highway`); `motorway_construction` = Motorway under construction (with kind `highway`); `primary` = Primary road (with kind `major_road`); `primary_construction` = Primary road under construction (with kind `major_road`); `secondary` = Secondary road (with kind `major_road`); `minor` = Minor road (with kind `minor_road`); `pedestrian` = Pedestrian zone (with kind `minor_road`); `service` = Service road (with kind `service`); `track` = Track (with kind `track`); `path` = Footpath (with kind `path`); `cycleway` = Cycleway (with kind `path`); `steps` = Steps (with kind `path`); `trail` = Hiking trail (with kind `path`) |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `oneway` | `boolean` | true for one-way roads (in digitised direction). |
| `ref` | `string` | Road number, e.g. A1 or B3. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |
| `structure` | `string` | Structure type: bridge or tunnel. Absent for at-grade segments. Values: `bridge` = Bridge; `tunnel` = Tunnel |

### rail_l

Geometry: **Line**, Zoom: **z6–16**, Layer: `rail_l`. Railways and ferry lines (lines).

| Field | Type | Description |
|---|---|---|
| `construction` | `boolean` | true if the line is still under construction. |
| `kind` | `string` | Rail type. Values: `rail` = Standard-gauge railway; `narrow_gauge` = Narrow-gauge railway; `rack` = Rack railway; `subway` = Subway; `tram` = Tram; `funicular` = Funicular; `ferry` = Ferry line |
| `kind_detail` | `string` | Importance of the line (standard-gauge, narrow-gauge and rack railways). Values: `main` = Main line; `branch` = Branch line; `siding` = Siding or spur |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `structure` | `string` | Structure type: bridge or tunnel. Absent for at-grade segments. Values: `bridge` = Bridge; `tunnel` = Tunnel |

### aerialway_l

Geometry: **Line**, Zoom: **z12–16**, Layer: `aerialway_l`. Cable cars and lifts (lines).

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Type of installation. Values: `cable_car` = Aerial tramway; `gondola` = Gondola lift; `chair_lift` = Chair lift; `drag_lift` = Drag lift; `goods` = Goods cableway (incl. avalanche-control cableway); `inclined_lift` = Inclined lift; `zip_line` = Zip line |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |

### transit_p

Geometry: **Point**, Zoom: **z10–16**, Layer: `transit_p`. Railway stations, stops and cable-car stations.

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Type of station. Values: `train_station` = Railway station; `halt` = Halt; `aerialway_station` = Cable-car station |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |

### airport_p

Geometry: **Point**, Zoom: **z8–16**, Layer: `airport_p`. Airports, airfields and heliports.

| Field | Type | Description |
|---|---|---|
| `iata` | `string` | IATA code (3 letters), if available. |
| `icao` | `string` | ICAO code (4 letters), if available. |
| `kind` | `string` | Type of aerodrome. Values: `airport` = Commercial airport; `airfield` = Airfield; `heliport` = Heliport; `military` = Military airfield |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |

## Water

### water_a

Geometry: **Polygon**, Zoom: **z6–16**, Layer: `water_a`. Standing water bodies (polygons).

| Field | Type | Description |
|---|---|---|
| `area_km2` | `number` | Area in square kilometres. |
| `kind` | `string` | Type of water body. Values: `lake` = Lake; `pond` = Pond; `reservoir` = Reservoir |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |

### water_l

Geometry: **Line**, Zoom: **z6–16**, Layer: `water_l`. Watercourses (lines).

| Field | Type | Description |
|---|---|---|
| `catchment_km2` | `number` | Catchment area in square kilometres. |
| `kind` | `string` | Type of watercourse. Values: `river` = River; `stream` = Stream; `canal` = Canal; `ditch` = Ditch |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |
| `structure` | `string` | Structure type: culverted or underground section. Absent for open sections. Values: `tunnel` = Tunnel, culvert |

### water_label_p

Geometry: **Point**, Zoom: **z6–16**, Layer: `water_label_p`. Label points for lakes and ponds.

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Type of water body. Values: `lake` = Lake; `pond` = Pond; `reservoir` = Reservoir |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |

### water_label_l

Geometry: **Line**, Zoom: **z10–16**, Layer: `water_label_l`. Label lines for rivers and canals.

| Field | Type | Description |
|---|---|---|
| `kind` | `string` | Type of watercourse. Values: `river` = River; `canal` = Canal |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |

## Boundaries

### boundary_l

Geometry: **Line**, Zoom: **z0–16**, Layer: `boundary_l`. Administrative boundaries (lines).

| Field | Type | Description |
|---|---|---|
| `admin_level` | `number` | Administrative level of the boundary (2 country, 4 state, 6 district, 8 municipality). Values: `2` = Country border; `4` = State border; `6` = District border; `8` = Municipality border |
| `kind` | `string` | Type of boundary. Values: `country` = Country; `state` = State; `district` = District; `municipality` = Municipality |
| `left` | `string` | Name of the administrative unit on the left of the line (not set for country borders). |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `right` | `string` | Name of the administrative unit on the right of the line (for country borders the adjoining area). |

### boundary_label_p

Geometry: **Point**, Zoom: **z4–16**, Layer: `boundary_label_p`. Label points for states, districts and municipalities.

| Field | Type | Description |
|---|---|---|
| `admin_level` | `number` | Administrative level. Values: `4` = State; `6` = District; `8` = Municipality |
| `code` | `string` | Official code of the unit. |
| `kind` | `string` | Type of administrative unit. Values: `state` = State; `district` = District; `municipality` = Municipality |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |

## Places and names

### place_p

Geometry: **Point**, Zoom: **z0–16**, Layer: `place_p`. Places and settlements.

| Field | Type | Description |
|---|---|---|
| `capital` | `string` | Capital role of the place; only set where applicable. Values: `country` = National capital; `state` = State capital; `district` = District seat |
| `kind` | `string` | Place category. Values: `city` = City; `town` = Market town; `village` = Village; `hamlet` = Hamlet; `suburb` = Suburb; `locality` = Isolated dwelling, farmstead |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `name_alt` | `string` | Alternative name, if available. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |

### natural_p

Geometry: **Point**, Zoom: **z8–16**, Layer: `natural_p`. Terrain names, peaks, passes, glaciers and named castles, palaces and small monuments.

| Field | Type | Description |
|---|---|---|
| `ele` | `number` | Elevation in metres. |
| `kind` | `string` | Type of feature. Values: `peak` = Peak; `saddle` = Pass, saddle; `ridge` = Mountain part, ridge; `valley` = Valley; `glacier` = Glacier; `mountain_range` = Mountain range; `area` = Area, region; `field_name` = Field name; `castle` = Castle; `palace` = Palace; `ruins` = Ruins; `tower` = Tower; `minor_monument` = Minor monument (wayside shrine, cross); `monument` = Monument; `other` = Other named feature |
| `kind_detail` | `string` | Additional display information. Values: `summit_label` = Mountain part, pass or valley shown like a peak; `sonstiges` = Other feature |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |

### poi_p

Geometry: **Point**, Zoom: **z13–16**, Layer: `poi_p`. Facilities and points of interest (points).

| Field | Type | Description |
|---|---|---|
| `ele` | `number` | Elevation in metres. |
| `kind` | `string` | Type of facility. Values: `bivouac` = Bivouac shelter; `alpine_hut` = Mountain hut; `inn` = Inn; `mine` = Mine; `cave` = Cave; `camp_site` = Camp site; `pharmacy` = Pharmacy; `police` = Police; `fire_station` = Fire station; `hospital` = Hospital; `clinic` = Clinic; `ambulance_station` = Ambulance station; `school` = School; `music_school` = Music school; `kindergarten` = Kindergarten; `university` = University; `survey_office` = Survey office; `district_office` = District office, city authority; `forestry` = Forestry; `townhall` = Town hall; `customs` = Customs; `swimming_pool` = Swimming pool; `swimming_area` = Beach, swimming area; `golf_course` = Golf course; `pitch` = Sports pitch; `stadium` = Stadium; `sports_centre` = Sports centre; `park` = Park; `zoo` = Zoo; `open_air_museum` = Open-air museum; `archaeological_site` = Archaeological site; `theatre` = Open-air theatre; `attraction` = Attraction, exhibition ground; `cemetery` = Cemetery; `visitor_centre` = Visitor centre; `ferry_terminal` = Ferry or ship terminal; `fuel` = Fuel station; `parking` = Parking; `rest_area` = Rest area; `toll_booth` = Toll booth; `helipad` = Helipad; `church` = Church; `chapel` = Chapel; `monastery` = Monastery, abbey; `summit_cross` = Summit cross; `place_of_worship` = Other place of worship; `museum` = Museum; `library` = Library; `drinking_water` = Drinking water; `toilets` = Public toilets; `playground` = Playground; `charging_station` = EV charging station; `defibrillator` = Defibrillator; `doctor` = Doctor; `lodging` = Lodging; `viewpoint` = Viewpoint; `market` = Market |
| `kind_detail` | `string` | Finer subdivision; only set for some facility types. Values: `public` = Public pharmacy (with kind `pharmacy`); `branch` = Branch pharmacy (with kind `pharmacy`); `hospital` = Hospital pharmacy (with kind `pharmacy`); `inspection` = Police station (with kind `police`); `motorway` = Motorway police (with kind `police`); `border` = Border police (with kind `police`); `traffic` = Traffic police (with kind `police`); `district_command` = District police command (with kind `police`); `city_command` = City police command (with kind `police`); `state_directorate` = State police directorate (with kind `police`); `volunteer` = Volunteer fire brigade (with kind `fire_station`); `professional` = Professional fire brigade (with kind `fire_station`); `company` = Company fire brigade (with kind `fire_station`); `district_command` = District fire command (with kind `fire_station`); `state_association` = State fire association (with kind `fire_station`); `acute` = Acute-care hospital (with kind `hospital`); `other` = Other hospital (with kind `hospital`); `outpatient` = Outpatient clinic (with kind `clinic`); `primary` = Primary school (with kind `school`); `secondary` = Secondary school (with kind `school`); `academic_secondary` = Academic secondary school (with kind `school`); `general` = Other general school (with kind `school`); `prevocational` = Prevocational school (with kind `school`); `special` = Special-needs school (with kind `school`); `vocational` = Vocational school (with kind `school`); `commercial` = Commercial school (with kind `school`); `technical` = Technical school (with kind `school`); `hospitality` = Hospitality and business school (with kind `school`); `social` = Social-care school (with kind `school`); `health` = Health and nursing school (with kind `school`); `agricultural` = Agricultural school (with kind `school`); `teacher_training` = Teacher training (with kind `school`); `other` = Other educational institution (with kind `school`); `district` = District administration (with kind `district_office`); `city` = City authority (with kind `district_office`); `forest_house` = Forest house (with kind `forestry`); `office` = Forestry office (with kind `forestry`); `hunting_lodge` = Hunting lodge (with kind `forestry`); `indoor` = Indoor pool (with kind `swimming_pool`); `indoor_outdoor` = Indoor and outdoor pool (with kind `swimming_pool`); `outdoor` = Outdoor pool (with kind `swimming_pool`); `beach` = Beach bath (with kind `swimming_area`); `tennis` = Tennis court (with kind `pitch`); `soccer` = Football pitch (with kind `pitch`); `ball_sports` = Ball sports facility (with kind `sports_centre`); `multi_sport` = Multi-sport facility (with kind `sports_centre`); `motor_sport` = Motor sports facility (with kind `sports_centre`); `winter_sport` = Winter sports facility (with kind `sports_centre`); `exhibition_ground` = Exhibition ground (with kind `attraction`); `zoo` = Zoo (with kind `zoo`); `wildlife_park` = Wildlife park (with kind `zoo`); `open_air` = Open-air theatre (with kind `theatre`); `forest` = Forest cemetery (with kind `cemetery`); `historic` = Historic cemetery (with kind `cemetery`); `pet` = Pet cemetery (with kind `cemetery`); `national_park` = National park visitor centre (with kind `visitor_centre`); `ferry` = Ferry (with kind `ferry_terminal`); `ship_station` = Ship station (with kind `ferry_terminal`); `picnic` = Picnic area (with kind `rest_area`); `service_area` = Service area (with kind `rest_area`); `eintuermig` = Church with one tower (with kind `church`); `mehrtuermig` = Church with several towers (with kind `church`); `eintuermig` = Monastery with one tower (with kind `monastery`); `mehrtuermig` = Monastery with several towers (with kind `monastery`); `fountain` = Drinking fountain (with kind `drinking_water`); `hydrant` = Drinking hydrant (with kind `drinking_water`); `garage` = Car park (with kind `parking`); `park_and_ride` = Park and ride (with kind `parking`); `synagogue` = Synagogue (with kind `place_of_worship`); `christian` = Christian congregation (with kind `place_of_worship`); `muslim` = Muslim congregation (with kind `place_of_worship`); `buddhist` = Buddhist congregation (with kind `place_of_worship`); `other` = Other congregation (with kind `place_of_worship`); `hotel` = Hotel (with kind `lodging`); `pension` = Guesthouse (with kind `lodging`) |
| `min_zoom` | `number` | Zoom level from which the feature is shown. |
| `name` | `string` | Name of the feature. |
| `sort_rank` | `number` | Display rank; lower number = more important or drawn further below. |

### housenumber_p

Geometry: **Point**, Zoom: **z16–16**, Layer: `housenumber_p`. House numbers.

| Field | Type | Description |
|---|---|---|
| `housenumber` | `string` | House number. |
| `kind` | `string` | Object the number refers to. Values: `building` = Building; `parcel` = Parcel |
| `street` | `string` | Street name. |

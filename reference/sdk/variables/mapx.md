# mapx

mapx-gl mapx

> `const` **mapx**: `object`

Namespace-Objekt: `mapx.map({...})`, `mapx.directions(...)`.

## Type declaration

### departures()

> **departures**: (`stop`, `o`) => `Promise`\<[`Departure`](/reference/sdk/interfaces/departure/)[]\>

Abfahrten (Soll-Fahrplan) an einer Haltestelle (`at:49:1320`) oder rund um eine Koordinate.

#### Parameters

• **stop**: `string` \| [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`DeparturesOptions`](/reference/sdk/interfaces/departuresoptions/) = `{}`

#### Returns

`Promise`\<[`Departure`](/reference/sdk/interfaces/departure/)[]\>

### DeparturesPopup

> **DeparturesPopup**: *typeof* [`DeparturesPopup`](/reference/sdk/classes/departurespopup/)

### directions()

> **directions**: (`profile`, `waypoints`, `o`) => `Promise`\<[`DirectionsResponse`](/reference/sdk/interfaces/directionsresponse/)\>

Routing (Mapbox-Directions-Antwort, Geometrie als GeoJSON). Profile: driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking.

#### Parameters

• **profile**: [`Profile`](/reference/sdk/type-aliases/profile/)

• **waypoints**: [`LL`](/reference/sdk/type-aliases/ll/)[]

• **o**: [`DirectionsOptions`](/reference/sdk/interfaces/directionsoptions/) = `{}`

#### Returns

`Promise`\<[`DirectionsResponse`](/reference/sdk/interfaces/directionsresponse/)\>

### elevation()

> **elevation**: (`points`, `model`) => `Promise`\<`number` \| `null` \| (`number` \| `null`)[]\>

Höhe (m) für einen oder mehrere Punkte; null außerhalb der Daten. `dsm` = Oberflächenmodell.

#### Parameters

• **points**: [`LL`](/reference/sdk/type-aliases/ll/) \| [`LL`](/reference/sdk/type-aliases/ll/)[]

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

#### Returns

`Promise`\<`number` \| `null` \| (`number` \| `null`)[]\>

### elevationProfile()

> **elevationProfile**: (`line`, `samples`, `model`, `signal`?) => `Promise`\<[`ElevationProfile`](/reference/sdk/interfaces/elevationprofile/)\>

Höhenprofil entlang einer Linie (2 bis 512 Stützpunkte).

#### Parameters

• **line**: [`LL`](/reference/sdk/type-aliases/ll/)[]

• **samples**: `number` = `100`

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

• **signal?**: `AbortSignal`

#### Returns

`Promise`\<[`ElevationProfile`](/reference/sdk/interfaces/elevationprofile/)\>

### ElevationProfileControl

> **ElevationProfileControl**: *typeof* [`ElevationProfileControl`](/reference/sdk/classes/elevationprofilecontrol/)

### geocode()

> **geocode**: (`q`, `o`) => `Promise`\<[`GeocodeFeature`](/reference/sdk/interfaces/geocodefeature/)[]\>

Geocoding: Adressen, Orte, Gipfel und mit `types: ['parcel']` Grundstücke (`KG 14105 Gst 102/3`).

#### Parameters

• **q**: `string`

• **o**: [`GeocodeOptions`](/reference/sdk/interfaces/geocodeoptions/) = `{}`

#### Returns

`Promise`\<[`GeocodeFeature`](/reference/sdk/interfaces/geocodefeature/)[]\>

### geojsonOverlay()

> **geojsonOverlay**: (`g`) => `string`

GeoJSON-Overlay (Simplestyle: stroke, stroke-width, fill, fill-opacity, marker-color).

#### Parameters

• **g**: `GeoJSON`\<`Geometry`, `GeoJsonProperties`\>

#### Returns

`string`

### isochrone()

> **isochrone**: (`profile`, `center`, `o`) => `Promise`\<`GeoJSON.FeatureCollection`\>

Isochronen als GeoJSON-FeatureCollection (größte Fläche zuerst).

#### Parameters

• **profile**: [`Profile`](/reference/sdk/type-aliases/profile/)

• **center**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`IsochroneOptions`](/reference/sdk/interfaces/isochroneoptions/) = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\>

### LayerControl

> **LayerControl**: *typeof* [`LayerControl`](/reference/sdk/classes/layercontrol/)

### map()

> **map**: (`o`) => `maplibregl.Map` = `createMap`

Karte mit mapx-Style anlegen. Registriert das pmtiles-Protokoll.

#### Parameters

• **o**: [`MapxOptions`](/reference/sdk/interfaces/mapxoptions/)

#### Returns

`maplibregl.Map`

### MapxError

> **MapxError**: *typeof* [`MapxError`](/reference/sdk/classes/mapxerror/)

### matrix()

> **matrix**: (`profile`, `points`, `o`) => `Promise`\<[`MatrixResponse`](/reference/sdk/interfaces/matrixresponse/)\>

Fahrzeit-/Distanzmatrix für mehrere Punkte in einer Anfrage.

#### Parameters

• **profile**: [`Profile`](/reference/sdk/type-aliases/profile/)

• **points**: [`LL`](/reference/sdk/type-aliases/ll/)[]

• **o**: [`MatrixOptions`](/reference/sdk/interfaces/matrixoptions/) = `{}`

#### Returns

`Promise`\<[`MatrixResponse`](/reference/sdk/interfaces/matrixresponse/)\>

### MeasureControl

> **MeasureControl**: *typeof* [`MeasureControl`](/reference/sdk/classes/measurecontrol/)

### optimization()

> **optimization**: (`body`, `signal`?) => `Promise`\<[`OptimizationResponse`](/reference/sdk/interfaces/optimizationresponse/)\>

Tourenplanung (Fahrzeuge, Aufträge, Zeitfenster). Höchstens 10 Fahrzeuge und 50 Aufträge; Zeiten in Sekunden.

#### Parameters

• **body**: [`OptimizationRequest`](/reference/sdk/interfaces/optimizationrequest/)

• **signal?**: `AbortSignal`

#### Returns

`Promise`\<[`OptimizationResponse`](/reference/sdk/interfaces/optimizationresponse/)\>

### parcel()

> **parcel**: (`point`, `o`) => `Promise`\<`GeoJSON.FeatureCollection` & `object`\>(`point`, `o`?) => `Promise`\<`GeoJSON.FeatureCollection` & `object`\>

Grundstück an einem Punkt (BEV DKM). Mit `dossier: true` liefert jedes Feature das volle Dossier.

#### Parameters

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`ParcelOptions`](/reference/sdk/interfaces/parceloptions/) & `object`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection` & `object`\>

#### Parameters

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o?**: [`ParcelOptions`](/reference/sdk/interfaces/parceloptions/) & `object`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection` & `object`\>

### parcelDossier()

> **parcelDossier**: (`id`, `signal`?) => `Promise`\<[`Dossier`](/reference/sdk/interfaces/dossier/)\>

Parzellen-Dossier: Fläche, Nutzung, Gebäude, Gelände, Adressen, nächste Haltestelle, Flugjahr, Static-Image-Link. ID aus `parcel_id`, z. B. `14143-473_1`.

#### Parameters

• **id**: `string`

• **signal?**: `AbortSignal`

#### Returns

`Promise`\<[`Dossier`](/reference/sdk/interfaces/dossier/)\>

### ParcelPopup

> **ParcelPopup**: *typeof* [`ParcelPopup`](/reference/sdk/classes/parcelpopup/)

### pin()

> **pin**: (`at`, `o`) => `string`

Marker-Overlay: `pin([16.37,48.2], { color: 'e11d48', size: 'l' })`.

#### Parameters

• **at**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o** = `{}`

• **o.color?**: `string`

• **o.size?**: `"s"` \| `"m"` \| `"l"`

#### Returns

`string`

### reverse()

> **reverse**: (`point`, `o`) => `Promise`\<[`GeocodeFeature`](/reference/sdk/interfaces/geocodefeature/)[]\>

Reverse Geocoding: nächstes Objekt zu einem Punkt (`lon,lat`).

#### Parameters

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`ReverseOptions`](/reference/sdk/interfaces/reverseoptions/) = `{}`

#### Returns

`Promise`\<[`GeocodeFeature`](/reference/sdk/interfaces/geocodefeature/)[]\>

### RouteControl

> **RouteControl**: *typeof* [`RouteControl`](/reference/sdk/classes/routecontrol/)

### SearchControl

> **SearchControl**: *typeof* [`SearchControl`](/reference/sdk/classes/searchcontrol/)

### searchRetrieve()

> **searchRetrieve**: (`id`, `o`) => `Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Point`\>\>

Search Box, Schritt 2: Koordinate und Details zu einem Vorschlag (`mapbox_id`).

#### Parameters

• **id**: `string`

• **o**: `Pick`\<[`SearchOptions`](/reference/sdk/interfaces/searchoptions/), `"proximity"` \| `"sessionToken"` \| `"signal"`\> = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Point`\>\>

### searchSuggest()

> **searchSuggest**: (`q`, `o`) => `Promise`\<[`Suggestion`](/reference/sdk/interfaces/suggestion/)[]\>

Search Box, Schritt 1: Vorschläge während der Eingabe. Dasselbe `sessionToken` für suggest und retrieve verwenden.

#### Parameters

• **q**: `string`

• **o**: [`SearchOptions`](/reference/sdk/interfaces/searchoptions/) = `{}`

#### Returns

`Promise`\<[`Suggestion`](/reference/sdk/interfaces/suggestion/)[]\>

### setAccessToken()

> **setAccessToken**: (`token`) => `void`

Zugriffstoken setzen (`mx_pk_…` im Browser, auf account.mapx.at erzeugt). Alle API-Anfragen senden ihn als `Authorization: Bearer`,
Karten (Style, Kacheln, Schriften, Sprites) als `access_token`-Query, Static-Image-URLs ebenfalls. Er geht nur an `mapx.at`-Hosts.

#### Parameters

• **token**: `undefined` \| `null` \| `string`

#### Returns

`void`

### setApiBase()

> **setApiBase**: (`url`) => `void`

Basis-URL der API ändern (z. B. Staging).

#### Parameters

• **url**: `string`

#### Returns

`void`

### setTerrain()

> **setTerrain**: (`map`, `on`, `exaggeration`) => `void`

3D-Gelände (neu) setzen; bleibt auch nach Style-Wechsel aktiv.

#### Parameters

• **map**: `Map$1`

• **on**: `boolean`

• **exaggeration**: `number` = `1.4`

#### Returns

`void`

### staticImageUrl()

> **staticImageUrl**: (`o`) => `string`

URL eines Static Image (für `<img src>`, E-Mail, Exposé). Kein Request, nur Zeichenketten. Mit gesetztem Token enthält sie `access_token` (öffentlicher `mx_pk_`-Token, an Origins gebunden).

#### Parameters

• **o**: [`StaticOptions`](/reference/sdk/interfaces/staticoptions/)

#### Returns

`string`

### stops()

> **stops**: (`o`) => `Promise`\<[`Stop`](/reference/sdk/interfaces/stop/)[]\>

Haltestellen im Fenster oder um einen Punkt.

#### Parameters

• **o**: [`StopsOptions`](/reference/sdk/interfaces/stopsoptions/) = `{}`

#### Returns

`Promise`\<[`Stop`](/reference/sdk/interfaces/stop/)[]\>

### styles

> **styles**: readonly [`"streets"`, `"streets-dawn"`, `"streets-dusk"`, `"streets-night"`, `"satellite"`, `"outdoors"`] = `STYLE_IDS`

### StyleSwitcherControl

> **StyleSwitcherControl**: *typeof* [`StyleSwitcherControl`](/reference/sdk/classes/styleswitchercontrol/)

### styleUrl()

> **styleUrl**: (`s`) => `string`

Kurzname -> Style-URL; volle URLs und Style-Objekte bleiben unverändert.

#### Parameters

• **s**: `string` \| `string` & `object`

#### Returns

`string`

### Terrain3DControl

> **Terrain3DControl**: *typeof* [`Terrain3DControl`](/reference/sdk/classes/terrain3dcontrol/)

### tilequery()

> **tilequery**: (`tilesets`, `point`, `o`) => `Promise`\<`GeoJSON.FeatureCollection`\>

Vektorkachel-Abfrage: nächste Objekte um einen Punkt (`radius` bis 2000 m). `tilesets` z. B. `['streets','cadastre']`.

#### Parameters

• **tilesets**: `string`[]

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`TilequeryOptions`](/reference/sdk/interfaces/tilequeryoptions/) = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\>

### trafficIncidents()

> **trafficIncidents**: (`o`) => `Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Geometry`, [`Incident`](/reference/sdk/interfaces/incident/)\>\>

Verkehrsmeldungen (ASFINAG/EVIS), gefiltert nach Fenster, Straße, Typ.

#### Parameters

• **o**: [`IncidentOptions`](/reference/sdk/interfaces/incidentoptions/) = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Geometry`, [`Incident`](/reference/sdk/interfaces/incident/)\>\>

### transformRequest()

> **transformRequest**: (`url`) => `object`

`transformRequest` für MapLibre: hängt den Token an mapx.at-Anfragen. Verträgt eigene Funktionen über `createMap({ transformRequest })`.

#### Parameters

• **url**: `string`

#### Returns

`object`

##### url

> **url**: `string`

### transitDirections()

> **transitDirections**: (`from`, `to`, `o`) => `Promise`\<`any`\>

ÖV-Verbindung (zu Fuß oder mit Rad zur Haltestelle). Zeit als ISO 8601 in Wiener Ortszeit.

#### Parameters

• **from**: [`LL`](/reference/sdk/type-aliases/ll/)

• **to**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`TransitOptions`](/reference/sdk/interfaces/transitoptions/) = `{}`

#### Returns

`Promise`\<`any`\>

### weatherCurrent()

> **weatherCurrent**: (`point`, `o`) => `Promise`\<[`WeatherStation`](/reference/sdk/interfaces/weatherstation/)[]\>

Aktuelle Messwerte der nächsten Stationen.

#### Parameters

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`Common`](/reference/sdk/interfaces/common/) & `object` = `{}`

#### Returns

`Promise`\<[`WeatherStation`](/reference/sdk/interfaces/weatherstation/)[]\>

### weatherForecast()

> **weatherForecast**: (`point`, `o`) => `Promise`\<`object`\>

Stundenvorhersage (AROME 1 km) für einen Punkt.

#### Parameters

• **point**: [`LL`](/reference/sdk/type-aliases/ll/)

• **o**: [`Common`](/reference/sdk/interfaces/common/) & `object` = `{}`

#### Returns

`Promise`\<`object`\>

##### hours

> **hours**: [`WeatherHour`](/reference/sdk/interfaces/weatherhour/)[]

##### model

> **model**: `string`

##### reference\_time

> **reference\_time**: `string`

### withToken()

> **withToken**: (`u`, `token`) => `string`

URL mit `access_token` versehen, wenn ein Token gesetzt ist, die URL zu mapx.at gehört und noch keinen Token trägt.
Bei `pmtiles://…/{z}/{x}/{y}` kommt er vor den Kachelpfad, bei Archiv-URLs ans Ende.

#### Parameters

• **u**: `string`

• **token**: `string` = `API.token`

#### Returns

`string`
# mapx

Source: https://docs.mapx.at/reference/sdk/variables/mapx/

> mapx-gl mapx

> `const` **mapx**: `object`

Namespace-Objekt: `mapx.map({...})`, `mapx.directions(...)`.

## Type declaration

### departures()

> **departures**: (`stop`, `o`) => `Promise`\<[`Departure`](https://docs.mapx.at/reference/sdk/interfaces/departure.md)[]\>

Abfahrten (Soll-Fahrplan) an einer Haltestelle (`at:49:1320`) oder rund um eine Koordinate.

#### Parameters

• **stop**: `string` \| [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`DeparturesOptions`](https://docs.mapx.at/reference/sdk/interfaces/departuresoptions.md) = `{}`

#### Returns

`Promise`\<[`Departure`](https://docs.mapx.at/reference/sdk/interfaces/departure.md)[]\>

### DeparturesPopup

> **DeparturesPopup**: *typeof* [`DeparturesPopup`](https://docs.mapx.at/reference/sdk/classes/departurespopup.md)

### directions()

> **directions**: (`profile`, `waypoints`, `o`) => `Promise`\<[`DirectionsResponse`](https://docs.mapx.at/reference/sdk/interfaces/directionsresponse.md)\>

Routing (Mapbox-Directions-Antwort, Geometrie als GeoJSON). Profile: driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking.

#### Parameters

• **profile**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)

• **waypoints**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **o**: [`DirectionsOptions`](https://docs.mapx.at/reference/sdk/interfaces/directionsoptions.md) = `{}`

#### Returns

`Promise`\<[`DirectionsResponse`](https://docs.mapx.at/reference/sdk/interfaces/directionsresponse.md)\>

### elevation()

> **elevation**: (`points`, `model`) => `Promise`\<`number` \| `null` \| (`number` \| `null`)[]\>

Höhe (m) für einen oder mehrere Punkte; null außerhalb der Daten. `dsm` = Oberflächenmodell.

#### Parameters

• **points**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md) \| [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

#### Returns

`Promise`\<`number` \| `null` \| (`number` \| `null`)[]\>

### elevationProfile()

> **elevationProfile**: (`line`, `samples`, `model`, `signal`?) => `Promise`\<[`ElevationProfile`](https://docs.mapx.at/reference/sdk/interfaces/elevationprofile.md)\>

Höhenprofil entlang einer Linie (2 bis 512 Stützpunkte).

#### Parameters

• **line**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **samples**: `number` = `100`

• **model**: `"dtm"` \| `"dsm"` = `'dtm'`

• **signal?**: `AbortSignal`

#### Returns

`Promise`\<[`ElevationProfile`](https://docs.mapx.at/reference/sdk/interfaces/elevationprofile.md)\>

### ElevationProfileControl

> **ElevationProfileControl**: *typeof* [`ElevationProfileControl`](https://docs.mapx.at/reference/sdk/classes/elevationprofilecontrol.md)

### geocode()

> **geocode**: (`q`, `o`) => `Promise`\<[`GeocodeFeature`](https://docs.mapx.at/reference/sdk/interfaces/geocodefeature.md)[]\>

Geocoding: Adressen, Orte, Gipfel und mit `types: ['parcel']` Grundstücke (`KG 14105 Gst 102/3`).

#### Parameters

• **q**: `string`

• **o**: [`GeocodeOptions`](https://docs.mapx.at/reference/sdk/interfaces/geocodeoptions.md) = `{}`

#### Returns

`Promise`\<[`GeocodeFeature`](https://docs.mapx.at/reference/sdk/interfaces/geocodefeature.md)[]\>

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

• **profile**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)

• **center**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`IsochroneOptions`](https://docs.mapx.at/reference/sdk/interfaces/isochroneoptions.md) = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\>

### LayerControl

> **LayerControl**: *typeof* [`LayerControl`](https://docs.mapx.at/reference/sdk/classes/layercontrol.md)

### map()

> **map**: (`o`) => `maplibregl.Map` = `createMap`

Karte mit mapx-Style anlegen. Registriert das pmtiles-Protokoll.

#### Parameters

• **o**: [`MapxOptions`](https://docs.mapx.at/reference/sdk/interfaces/mapxoptions.md)

#### Returns

`maplibregl.Map`

### MapxError

> **MapxError**: *typeof* [`MapxError`](https://docs.mapx.at/reference/sdk/classes/mapxerror.md)

### matrix()

> **matrix**: (`profile`, `points`, `o`) => `Promise`\<[`MatrixResponse`](https://docs.mapx.at/reference/sdk/interfaces/matrixresponse.md)\>

Fahrzeit-/Distanzmatrix für mehrere Punkte in einer Anfrage.

#### Parameters

• **profile**: [`Profile`](https://docs.mapx.at/reference/sdk/type-aliases/profile.md)

• **points**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)[]

• **o**: [`MatrixOptions`](https://docs.mapx.at/reference/sdk/interfaces/matrixoptions.md) = `{}`

#### Returns

`Promise`\<[`MatrixResponse`](https://docs.mapx.at/reference/sdk/interfaces/matrixresponse.md)\>

### MeasureControl

> **MeasureControl**: *typeof* [`MeasureControl`](https://docs.mapx.at/reference/sdk/classes/measurecontrol.md)

### optimization()

> **optimization**: (`body`, `signal`?) => `Promise`\<[`OptimizationResponse`](https://docs.mapx.at/reference/sdk/interfaces/optimizationresponse.md)\>

Tourenplanung (Fahrzeuge, Aufträge, Zeitfenster). Höchstens 10 Fahrzeuge und 50 Aufträge; Zeiten in Sekunden.

#### Parameters

• **body**: [`OptimizationRequest`](https://docs.mapx.at/reference/sdk/interfaces/optimizationrequest.md)

• **signal?**: `AbortSignal`

#### Returns

`Promise`\<[`OptimizationResponse`](https://docs.mapx.at/reference/sdk/interfaces/optimizationresponse.md)\>

### parcel()

> **parcel**: (`point`, `o`) => `Promise`\<`GeoJSON.FeatureCollection` & `object`\>(`point`, `o`?) => `Promise`\<`GeoJSON.FeatureCollection` & `object`\>

Grundstück an einem Punkt (BEV DKM). Mit `dossier: true` liefert jedes Feature das volle Dossier.

#### Parameters

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`ParcelOptions`](https://docs.mapx.at/reference/sdk/interfaces/parceloptions.md) & `object`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection` & `object`\>

#### Parameters

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o?**: [`ParcelOptions`](https://docs.mapx.at/reference/sdk/interfaces/parceloptions.md) & `object`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection` & `object`\>

### parcelDossier()

> **parcelDossier**: (`id`, `signal`?) => `Promise`\<[`Dossier`](https://docs.mapx.at/reference/sdk/interfaces/dossier.md)\>

Parzellen-Dossier: Fläche, Nutzung, Gebäude, Gelände, Adressen, nächste Haltestelle, Flugjahr, Static-Image-Link. ID aus `parcel_id`, z. B. `14143-473_1`.

#### Parameters

• **id**: `string`

• **signal?**: `AbortSignal`

#### Returns

`Promise`\<[`Dossier`](https://docs.mapx.at/reference/sdk/interfaces/dossier.md)\>

### ParcelPopup

> **ParcelPopup**: *typeof* [`ParcelPopup`](https://docs.mapx.at/reference/sdk/classes/parcelpopup.md)

### pin()

> **pin**: (`at`, `o`) => `string`

Marker-Overlay: `pin([16.37,48.2], { color: 'e11d48', size: 'l' })`.

#### Parameters

• **at**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o** = `{}`

• **o.color?**: `string`

• **o.size?**: `"s"` \| `"m"` \| `"l"`

#### Returns

`string`

### reverse()

> **reverse**: (`point`, `o`) => `Promise`\<[`GeocodeFeature`](https://docs.mapx.at/reference/sdk/interfaces/geocodefeature.md)[]\>

Reverse Geocoding: nächstes Objekt zu einem Punkt (`lon,lat`).

#### Parameters

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`ReverseOptions`](https://docs.mapx.at/reference/sdk/interfaces/reverseoptions.md) = `{}`

#### Returns

`Promise`\<[`GeocodeFeature`](https://docs.mapx.at/reference/sdk/interfaces/geocodefeature.md)[]\>

### RouteControl

> **RouteControl**: *typeof* [`RouteControl`](https://docs.mapx.at/reference/sdk/classes/routecontrol.md)

### SearchControl

> **SearchControl**: *typeof* [`SearchControl`](https://docs.mapx.at/reference/sdk/classes/searchcontrol.md)

### searchRetrieve()

> **searchRetrieve**: (`id`, `o`) => `Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Point`\>\>

Search Box, Schritt 2: Koordinate und Details zu einem Vorschlag (`mapbox_id`).

#### Parameters

• **id**: `string`

• **o**: `Pick`\<[`SearchOptions`](https://docs.mapx.at/reference/sdk/interfaces/searchoptions.md), `"proximity"` \| `"sessionToken"` \| `"signal"`\> = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Point`\>\>

### searchSuggest()

> **searchSuggest**: (`q`, `o`) => `Promise`\<[`Suggestion`](https://docs.mapx.at/reference/sdk/interfaces/suggestion.md)[]\>

Search Box, Schritt 1: Vorschläge während der Eingabe. Dasselbe `sessionToken` für suggest und retrieve verwenden.

#### Parameters

• **q**: `string`

• **o**: [`SearchOptions`](https://docs.mapx.at/reference/sdk/interfaces/searchoptions.md) = `{}`

#### Returns

`Promise`\<[`Suggestion`](https://docs.mapx.at/reference/sdk/interfaces/suggestion.md)[]\>

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

• **o**: [`StaticOptions`](https://docs.mapx.at/reference/sdk/interfaces/staticoptions.md)

#### Returns

`string`

### stops()

> **stops**: (`o`) => `Promise`\<[`Stop`](https://docs.mapx.at/reference/sdk/interfaces/stop.md)[]\>

Haltestellen im Fenster oder um einen Punkt.

#### Parameters

• **o**: [`StopsOptions`](https://docs.mapx.at/reference/sdk/interfaces/stopsoptions.md) = `{}`

#### Returns

`Promise`\<[`Stop`](https://docs.mapx.at/reference/sdk/interfaces/stop.md)[]\>

### styles

> **styles**: readonly [`"streets"`, `"streets-dawn"`, `"streets-dusk"`, `"streets-night"`, `"satellite"`, `"outdoors"`] = `STYLE_IDS`

### StyleSwitcherControl

> **StyleSwitcherControl**: *typeof* [`StyleSwitcherControl`](https://docs.mapx.at/reference/sdk/classes/styleswitchercontrol.md)

### styleUrl()

> **styleUrl**: (`s`) => `string`

Kurzname -> Style-URL; volle URLs und Style-Objekte bleiben unverändert.

#### Parameters

• **s**: `string` \| `string` & `object`

#### Returns

`string`

### Terrain3DControl

> **Terrain3DControl**: *typeof* [`Terrain3DControl`](https://docs.mapx.at/reference/sdk/classes/terrain3dcontrol.md)

### tilequery()

> **tilequery**: (`tilesets`, `point`, `o`) => `Promise`\<`GeoJSON.FeatureCollection`\>

Vektorkachel-Abfrage: nächste Objekte um einen Punkt (`radius` bis 2000 m). `tilesets` z. B. `['streets','cadastre']`.

#### Parameters

• **tilesets**: `string`[]

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`TilequeryOptions`](https://docs.mapx.at/reference/sdk/interfaces/tilequeryoptions.md) = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\>

### trafficIncidents()

> **trafficIncidents**: (`o`) => `Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Geometry`, [`Incident`](https://docs.mapx.at/reference/sdk/interfaces/incident.md)\>\>

Verkehrsmeldungen (ASFINAG/EVIS), gefiltert nach Fenster, Straße, Typ.

#### Parameters

• **o**: [`IncidentOptions`](https://docs.mapx.at/reference/sdk/interfaces/incidentoptions.md) = `{}`

#### Returns

`Promise`\<`GeoJSON.FeatureCollection`\<`GeoJSON.Geometry`, [`Incident`](https://docs.mapx.at/reference/sdk/interfaces/incident.md)\>\>

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

• **from**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **to**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`TransitOptions`](https://docs.mapx.at/reference/sdk/interfaces/transitoptions.md) = `{}`

#### Returns

`Promise`\<`any`\>

### weatherCurrent()

> **weatherCurrent**: (`point`, `o`) => `Promise`\<[`WeatherStation`](https://docs.mapx.at/reference/sdk/interfaces/weatherstation.md)[]\>

Aktuelle Messwerte der nächsten Stationen.

#### Parameters

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`Common`](https://docs.mapx.at/reference/sdk/interfaces/common.md) & `object` = `{}`

#### Returns

`Promise`\<[`WeatherStation`](https://docs.mapx.at/reference/sdk/interfaces/weatherstation.md)[]\>

### weatherForecast()

> **weatherForecast**: (`point`, `o`) => `Promise`\<`object`\>

Stundenvorhersage (AROME 1 km) für einen Punkt.

#### Parameters

• **point**: [`LL`](https://docs.mapx.at/reference/sdk/type-aliases/ll.md)

• **o**: [`Common`](https://docs.mapx.at/reference/sdk/interfaces/common.md) & `object` = `{}`

#### Returns

`Promise`\<`object`\>

##### hours

> **hours**: [`WeatherHour`](https://docs.mapx.at/reference/sdk/interfaces/weatherhour.md)[]

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

# SDK helpers for the APIs

One typed function per API: search, geocoding, routing, matrix, optimization, parcels, elevation, traffic, transit, weather and static images.

Every helper is a plain `async` function that returns typed JSON and throws a [`MapxError`](/guides/sdk/errors/) on failure. Coordinates are `[lng, lat]`. Pass `signal` (an `AbortSignal`) to cancel a request. All examples run in the browser console on any page that has loaded `mapx` ([install](/guides/sdk/)) and set a token with `mapx.setAccessToken("YOUR_TOKEN")`.

## Search and geocoding

```js
// Search Box: suggest while typing, retrieve after the click. One session token per search.
const sessionToken = crypto.randomUUID();
const hits = await mapx.searchSuggest("Stephansplatz", { sessionToken, proximity: [16.37, 48.21], types: ["poi", "street"], limit: 5 });
const place = await mapx.searchRetrieve(hits[0].mapbox_id, { sessionToken });

// Geocoding, also parcels: "KG 14143 Gst 473/1"
const parcels = await mapx.geocode("14143/473/1", { types: ["parcel"], limit: 3 });

// Reverse geocoding: nearest address, street or place
const [nearest] = await mapx.reverse([16.3725, 48.2085], { types: ["address"], limit: 1 });
```

## Routing

```js
const vienna = [16.3725, 48.2085], melk = [15.3294, 48.2272];

// All profiles: driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking
const car = await mapx.directions("driving", [vienna, melk], { alternatives: true, exclude: ["toll"], annotations: ["duration", "maxspeed"] });
const truck = await mapx.directions("driving-truck", [vienna, melk], { truck: { weight: 40, height: 4, width: 2.55, type: "semi" } });
const mtb = await mapx.directions("cycling-mtb", [vienna, melk], { steps: true, language: "en" });
const bus = await mapx.transitDirections(vienna, [16.4, 48.19], { departAt: "2026-11-09T08:00", routes: 2 });

console.log(car.routes[0].distance, truck.routes[0].truck, mtb.routes[0].legs[0].steps.length);
```

`exclude` (`motorway`, `toll`) works for `driving`, `driving-traffic` and `driving-truck`, `truck` only for `driving-truck`. The server validates both: a wrong combination throws a `MapxError` with `code: "bad-request"`. The `ferry` exclusion is not part of the typed options yet; use the [Directions API](/api/directions/) for it.

```js
// Reachability and travel times
const iso = await mapx.isochrone("cycling", melk, { minutes: [10, 20, 30] });          // GeoJSON polygons
const m = await mapx.matrix("driving", [vienna, melk, [14.2858, 48.3069]]);           // m.durations[i][j] in seconds

// Tour planning: one vehicle, two jobs
const tour = await mapx.optimization({
  vehicles: [{ id: 1, start: vienna, end: vienna }],
  services: [{ id: 1, location: melk, service: 300 }, { id: 2, location: [14.2858, 48.3069], service: 300 }],
});
console.log(tour.routes[0].steps.map((s) => s.type));
```

## Parcels and tilequery

```js
const { features } = await mapx.parcel([15.3294, 48.2272]);                // parcel polygon, number, area_m2
const dossier = await mapx.parcelDossier(features[0].properties.parcel_id); // land use, buildings, terrain, addresses, nearest stop
const near = await mapx.tilequery(["streets"], [16.3725, 48.2085], { radius: 50, layers: ["road_l"], limit: 3 });
```

`parcel(point, { dossier: true })` returns the dossier for every parcel at the point in one request.

## Elevation

```js
const h = await mapx.elevation([12.694, 47.074]);                 // 3726.3 (m), null outside the data
const hs = await mapx.elevation([[12.694, 47.074], [12.7, 47.08]], "dsm");
const p = await mapx.elevationProfile([[12.694, 47.074], [12.75, 47.1]], 100);
console.log(p.ascent, p.descent, p.min, p.max, p.length);
```

## Traffic, transit, weather

```js
const bbox = [16.0, 48.0, 16.6, 48.4];
const incidents = await mapx.trafficIncidents({ bbox, active: true, limit: 20 });   // GeoJSON features with text, road, type

const near = await mapx.stops({ near: [16.3725, 48.2085], radius: 300, limit: 3 });
const board = await mapx.departures(near[0].id, { limit: 5 });                       // scheduled timetable
console.log(board.map((d) => `${d.time.slice(11, 16)} ${d.line.name} ${d.headsign}`));

const now = await mapx.weatherCurrent([16.37, 48.2], { limit: 1 });                  // nearest station
const next = await mapx.weatherForecast([16.37, 48.2], { hours: 6 });                // hourly model values
```

## Static image URL

`staticImageUrl` builds the URL without a request. Use it in `<img>`, e-mails and PDFs.

```js
const url = mapx.staticImageUrl({
  style: "satellite", center: [15.3294, 48.2272], zoom: 17, width: 800, height: 500, format: "webp", retina: true,
  overlays: [mapx.pin([15.3294, 48.2272], { color: "e11d48", size: "l" })],
});
document.querySelector("img").src = url;
```

Use `mapx.geojsonOverlay(feature)` to draw a polygon (for example a parcel) with Simplestyle properties, and leave `center` and `zoom` out to fit the overlays (`auto`).

## Other API host

`mapx.setApiBase("https://staging.example.com/v1")` redirects all helpers, for tests or a mirror.

## Next

- [Controls and popups](/guides/sdk/controls/)
- [Errors and retries](/guides/sdk/errors/)
- [SDK reference](/reference/sdk/) with all types
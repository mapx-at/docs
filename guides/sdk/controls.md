# SDK controls and popups

Controls are added with `map.addControl(control, position)`. Popups attach to the map with `new Popup(map, options)`. Texts are German when the page language or browser is German, otherwise English.

## RouteControl

A route planner: click start and destination, switch the profile, see distance and time, up to two alternatives for cars.

```js
const route = new mapx.RouteControl({
  profile: "driving-truck",                  // start profile; default: first of `profiles`
  profiles: ["driving", "driving-truck", "cycling", "cycling-mtb", "walking"],
  onRoute: (response, request) => console.log(request.profile, response.routes[0].distance),
});
map.addControl(route, "top-right");
await route.setWaypoints([16.3725, 48.2085], [15.3294, 48.2272]); // set both points from code
```

For cars the panel offers *avoid tolls* and *avoid motorways* (`exclude`), for trucks weight, height, width and length. `route.last` holds the last response. [Live example](/examples/sdk-controls/).

## LayerControl

Switches whole layer groups on and off. The groups come from the style metadata (`mapx:groups`), so it works with every mapx style and shows new groups without a code change. Overlays from `mapx:overlays` (for example traffic) are offered as well when you do not restrict `groups`.

```js
map.addControl(new mapx.LayerControl({
  groups: ["buildings", "parcels", "relief", "bike_routes"],
  labels: { bike_routes: "Cycle routes", relief: "Hillshade and contours" },
}), "top-right");
```

## Search, styles, terrain, measuring

```js
map.addControl(new mapx.SearchControl({ placeholder: "Search" }), "top-left");        // /v1/geocode
map.addControl(new mapx.StyleSwitcherControl({ styles: ["streets", "satellite", "outdoors"] }), "top-right");
map.addControl(new mapx.Terrain3DControl(), "top-right");
map.addControl(new mapx.MeasureControl(), "top-right");                                 // length and area
map.addControl(new mapx.ElevationProfileControl(), "top-right");                        // draw a line, get the profile
```

## ParcelPopup

A click shows the parcel at that point: number, area, cadastral municipality, and with `dossier: true` (default) buildings and elevation range. The outline is highlighted.

```js
new mapx.ParcelPopup(map, { dossier: true, onParcel: (p) => console.log(p.properties.parcel_id) });
```

If the dossier is not available, the popup falls back to the basic parcel data.

## DeparturesPopup

A click finds the nearest stop within `radius` metres and lists the next departures of the scheduled timetable.

```js
const departures = new mapx.DeparturesPopup(map, { limit: 8, radius: 150 });
departures.open("at:49:1320"); // or by stop id, or [lng, lat]
```

[Live example](/examples/departures/).

## Combine popups and tools

Both popups react to every map click. When you use them together with `RouteControl`, `MeasureControl` or another click tool, create them on demand and call `remove()` while the tool is active:

```js
let parcels = new mapx.ParcelPopup(map);
document.querySelector("#tools").onchange = (e) => (e.target.checked ? parcels.remove() : (parcels = new mapx.ParcelPopup(map)));
```

## Next

- [SDK helpers](/guides/sdk/helpers/)
- [Errors and retries](/guides/sdk/errors/)
- [SDK reference](/reference/sdk/)


# Municipalities

Parcels, address validation, transit departures and an embedded map for municipal websites and administration tools.

Municipal tools work with parcels and addresses, need to be understood by residents and should not require a license. The pieces below combine the cadastre, the address register, the timetable and the map for the web page of a municipality.

## Parcels

The cadastre is available as a vector tileset for drawing and as an API for the parcel at a point.

```js
const map = mapx.map({ container: "map", style: "satellite-streets", center: [15.3294, 48.2272], zoom: 17 });
new mapx.ParcelPopup(map); // click: number, area, cadastral municipality, buildings
```

Search for a parcel by its number (the cadastral municipality is part of the identity):

```bash
curl -s "https://api.mapx.at/v1/geocode?q=KG%2014143%20Gst%20473/1&types=parcel&limit=1" | jq '.features[0] | {place_name, center, id: .properties.parcel_id}'
```

The boundaries come from the BEV cadastral map (DKM). The area is computed from the boundary lines and deviates from the register by per mille. State this where the data is shown: for orientation, not legally binding.

## Check addresses

Forms for registrations, applications and reports should accept only existing addresses. `address/validate` compares the input with the address register and returns `match`, `corrected`, `ambiguous` or `not_found` with the corrected address and coordinates.

```js
const q = new URLSearchParams({ street: "Haubtplatz", housenumber: "1", postcode: "3390", place: "Melk" });
const r = await (await fetch(`https://api.mapx.at/v1/address/validate?${q}`)).json();
console.log(r.status, r.deviations.map((d) => `${d.field}: ${d.input} > ${d.expected}`), r.address?.full_address ?? r.candidates?.map((c) => c.full_address));
```

While the person types, offer [address autofill](/examples/autofill/); for lists with many addresses use `POST /v1/geocode/batch` ([Search](/guides/search/)).

## Departures at the town hall

Residents ask "When does the next bus go?". The timetable (scheduled, no real-time data) is one request away:

```js
const [stop] = await mapx.stops({ near: [15.3294, 48.2272], radius: 400, limit: 1 });
const board = await mapx.departures(stop.id, { limit: 6 });
document.querySelector("#abfahrten").innerHTML = board.map((d) => `<li>${d.time.slice(11, 16)} ${d.line.name} → ${d.headsign}</li>`).join("");
```

Or add `new mapx.DeparturesPopup(map)` to a map. See [Transit departures](/guides/transit/) and the [example](/examples/departures/). Timetable data: ÖBB-Personenverkehr AG, data source Mobilitätsverbünde Österreich (CC BY 4.0).

## Embed a map in the municipal website

A map needs one HTML file, MapLibre and the style URL, no key and no build ([Embed a map](/examples/embed-map/)). Host the page with your content management system, or embed the file as `<iframe>`:

```html
<iframe src="/karte.html" title="Map of our municipality" style="width:100%;height:480px;border:0" loading="lazy"></iframe>
```

For a plain picture without JavaScript (print, newsletter, page without scripts) use a [static image](/guides/static-images/):

```html
<img alt="Location of the town hall" width="600" height="400"
     src="https://api.mapx.at/v1/static/streets/pin-l%2Be11d48(15.3294,48.2272)/15.3294,48.2272,16/600x400.png">
```

Show your own data (playgrounds, collection points, closures) as [GeoJSON on the map](/examples/geojson-overlay/). Keep the attribution of the data visible ([Attribution](/reference/attribution/)).

## Next steps

- [Parcels](/guides/parcels/) and [Search](/guides/search/)
- [Transit departures](/guides/transit/)
- [Style the basemap](/guides/style-the-basemap/) in the colours of your municipality
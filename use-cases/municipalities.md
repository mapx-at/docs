# Maps for municipalities and administration

Source: https://docs.mapx.at/use-cases/municipalities/

> Address validation, transit departures and an embedded map for municipal websites and administration tools.

Municipal tools work with addresses and need to be understood by residents. mapx is free with an API token ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)); replace `YOUR_TOKEN` with your own. The pieces below combine the address register, the timetable and the map for the web page of a municipality.

## Check addresses

Forms for registrations, applications and reports should accept only existing addresses. `address/validate` compares the input with the address register and returns `match`, `corrected`, `ambiguous` or `not_found` with the corrected address and coordinates.

```js
const q = new URLSearchParams({ street: "Haubtplatz", housenumber: "1", postcode: "3390", place: "Melk" });
const r = await (await fetch(`https://api.mapx.at/v1/address/validate?${q}&access_token=YOUR_TOKEN`)).json();
console.log(r.status, r.deviations.map((d) => `${d.field}: ${d.input} > ${d.expected}`), r.address?.full_address ?? r.candidates?.map((c) => c.full_address));
```

While the person types, offer [address autofill](https://docs.mapx.at/examples/autofill.md); for lists with many addresses use `POST /v1/geocode/batch` ([Search](https://docs.mapx.at/guides/search.md)), secret token (`mx_sk_…`) from a server only, otherwise `403 pk-not-allowed`.

## Departures at the town hall

Residents ask "When does the next bus go?". The timetable (scheduled, no real-time data) is one request away:

```js
const H = { headers: { Authorization: "Bearer YOUR_TOKEN" } };
const { stops: [stop] } = await (await fetch("https://api.mapx.at/v1/stops?near=15.3294,48.2272&radius=400&limit=1", H)).json();
const { departures } = await (await fetch(`https://api.mapx.at/v1/departures?stop=${encodeURIComponent(stop.id)}&limit=6`, H)).json();
document.querySelector("#abfahrten").replaceChildren(...departures.map((d) => Object.assign(document.createElement("li"), { textContent: `${d.time.slice(11, 16)} ${d.line.name} → ${d.headsign}` })));
```

On a map, the [example](https://docs.mapx.at/examples/departures.md) does the same with a click. See [Transit departures](https://docs.mapx.at/guides/transit.md). Timetable data: ÖBB-Personenverkehr AG, data source Mobilitätsverbünde Österreich (CC BY 4.0).

## Embed a map in the municipal website

A map needs one HTML file, MapLibre and the style URL, a free public token and no build ([Embed a map](https://docs.mapx.at/examples/embed-map.md)). Host the page with your content management system, or embed the file as `<iframe>`:

```html
<iframe src="/karte.html" title="Map of our municipality" style="width:100%;height:480px;border:0" loading="lazy"></iframe>
```

For a plain picture without JavaScript (print, newsletter, page without scripts) use a [static image](https://docs.mapx.at/guides/static-images.md):

```html
<img alt="Location of the town hall" width="600" height="400"
     src="https://api.mapx.at/v1/static/streets/pin-l%2Be11d48(15.3294,48.2272)/15.3294,48.2272,16/600x400.png?access_token=YOUR_TOKEN">
```

Show your own data (playgrounds, collection points, closures) as [GeoJSON on the map](https://docs.mapx.at/examples/geojson-overlay.md). The `<img>` URL carries the token: use a public token (`mx_pk_…`) with your website as allowed origin; render images for e-mails or PDFs on your server with a secret token (`mx_sk_…`). Keep the attribution of the data visible ([Attribution](https://docs.mapx.at/reference/attribution.md)).

## Next steps

- [Search](https://docs.mapx.at/guides/search.md)
- [Transit departures](https://docs.mapx.at/guides/transit.md)
- [Style the basemap](https://docs.mapx.at/guides/style-the-basemap.md) in the colours of your municipality

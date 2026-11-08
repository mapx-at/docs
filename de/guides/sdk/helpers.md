# SDK-Helfer für die APIs

Eine typisierte Funktion je API: Suche, Geocoding, Routing, Matrix, Optimization, Grundstücke, Höhe, Verkehr, ÖV, Wetter und statische Bilder.

Jeder Helfer ist eine einfache `async`-Funktion, die typisiertes JSON liefert und bei Fehlern einen [`MapxError`](/de/guides/sdk/errors/) wirft. Koordinaten sind `[lng, lat]`. Mit `signal` (ein `AbortSignal`) lässt sich eine Anfrage abbrechen. Alle Beispiele laufen in der Browser-Konsole jeder Seite, die `mapx` geladen hat ([Installation](/de/guides/sdk/)).

## Suche und Geocoding

```js
// Search Box: suggest beim Tippen, retrieve nach dem Klick. Ein Session-Token je Suche.
const sessionToken = crypto.randomUUID();
const hits = await mapx.searchSuggest("Stephansplatz", { sessionToken, proximity: [16.37, 48.21], types: ["poi", "street"], limit: 5 });
const place = await mapx.searchRetrieve(hits[0].mapbox_id, { sessionToken });

// Geocoding, auch Grundstücke: "KG 14105 Gst 102/3"
const parcels = await mapx.geocode("14105/102/3", { types: ["parcel"], limit: 3 });

// Reverse Geocoding: nächste Adresse, Straße oder Ort
const [nearest] = await mapx.reverse([16.3725, 48.2085], { types: ["address"], limit: 1 });
```

## Routing

```js
const vienna = [16.3725, 48.2085], melk = [15.3294, 48.2272];

// Alle Profile: driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking
const car = await mapx.directions("driving", [vienna, melk], { alternatives: true, exclude: ["toll"], annotations: ["duration", "maxspeed"] });
const truck = await mapx.directions("driving-truck", [vienna, melk], { truck: { weight: 40, height: 4, width: 2.55, type: "semi" } });
const mtb = await mapx.directions("cycling-mtb", [vienna, melk], { steps: true, language: "de" });
const bus = await mapx.transitDirections(vienna, [16.4, 48.19], { departAt: "2026-11-09T08:00", routes: 2 });

console.log(car.routes[0].distance, truck.routes[0].truck, mtb.routes[0].legs[0].steps.length);
```

`exclude` gilt nur für das Profil `driving`, `truck` nur für `driving-truck`. Beides wird geprüft: eine falsche Kombination wirft einen `MapxError` mit `code: "bad-request"`.

```js
// Erreichbarkeit und Fahrzeiten
const iso = await mapx.isochrone("cycling", melk, { minutes: [10, 20, 30] });          // GeoJSON-Flächen
const m = await mapx.matrix("driving", [vienna, melk, [14.2858, 48.3069]]);           // m.durations[i][j] in Sekunden

// Tourenplanung: ein Fahrzeug, zwei Aufträge
const tour = await mapx.optimization({
  vehicles: [{ id: 1, start: vienna, end: vienna }],
  services: [{ id: 1, location: melk, service: 300 }, { id: 2, location: [14.2858, 48.3069], service: 300 }],
});
console.log(tour.routes[0].steps.map((s) => s.type));
```

## Grundstücke und Tilequery

```js
const { features } = await mapx.parcel([15.3294, 48.2272]);                // Grundstücksfläche, Nummer, area_m2
const dossier = await mapx.parcelDossier(features[0].properties.parcel_id); // Nutzung, Gebäude, Gelände, Adressen, nächste Haltestelle
const near = await mapx.tilequery(["streets"], [16.3725, 48.2085], { radius: 50, layers: ["road_l"], limit: 3 });
```

`parcel(point, { dossier: true })` liefert das Dossier zu jedem Grundstück am Punkt in einer Anfrage.

## Höhe

```js
const h = await mapx.elevation([12.694, 47.074]);                 // 3726.3 (m), null außerhalb der Daten
const hs = await mapx.elevation([[12.694, 47.074], [12.7, 47.08]], "dsm");
const p = await mapx.elevationProfile([[12.694, 47.074], [12.75, 47.1]], 100);
console.log(p.ascent, p.descent, p.min, p.max, p.length);
```

## Verkehr, ÖV, Wetter

```js
const bbox = [16.0, 48.0, 16.6, 48.4];
const incidents = await mapx.trafficIncidents({ bbox, active: true, limit: 20 });   // GeoJSON-Features mit text, road, type

const near = await mapx.stops({ near: [16.3725, 48.2085], radius: 300, limit: 3 });
const board = await mapx.departures(near[0].id, { limit: 5 });                       // Soll-Fahrplan
console.log(board.map((d) => `${d.time.slice(11, 16)} ${d.line.name} ${d.headsign}`));

const now = await mapx.weatherCurrent([16.37, 48.2], { limit: 1 });                  // nächste Station
const next = await mapx.weatherForecast([16.37, 48.2], { hours: 6 });                // stündliche Modellwerte
```

## URL eines statischen Bildes

`staticImageUrl` baut die URL ohne Anfrage. Sie passt in `<img>`, E-Mails und PDFs.

```js
const url = mapx.staticImageUrl({
  style: "satellite", center: [15.3294, 48.2272], zoom: 17, width: 800, height: 500, format: "webp", retina: true,
  overlays: [mapx.pin([15.3294, 48.2272], { color: "e11d48", size: "l" })],
});
document.querySelector("img").src = url;
```

Mit `mapx.geojsonOverlay(feature)` lässt sich ein Polygon (etwa ein Grundstück) mit Simplestyle-Eigenschaften zeichnen; ohne `center` und `zoom` passt das Bild auf die Overlays (`auto`).

## Anderer API-Host

`mapx.setApiBase("https://staging.example.com/v1")` leitet alle Helfer um, für Tests oder einen Spiegel.

## Weiter

- [Controls und Popups](/de/guides/sdk/controls/)
- [Fehler und Wiederholungen](/de/guides/sdk/errors/)
- [SDK-Referenz](/reference/sdk/) mit allen Typen
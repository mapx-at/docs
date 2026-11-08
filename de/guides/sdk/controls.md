# SDK-Controls und Popups

RouteControl, LayerControl, SearchControl, ParcelPopup und DeparturesPopup: fertige Kartenwerkzeuge in je einer Zeile.

Controls kommen mit `map.addControl(control, position)` auf die Karte. Popups hängen sich mit `new Popup(map, options)` an die Karte. Die Texte sind deutsch, wenn Seite oder Browser deutsch sind, sonst englisch.

## RouteControl

Ein Routenplaner: Start und Ziel anklicken, Profil wechseln, Strecke und Zeit sehen, für Pkw bis zu zwei Alternativen.

```js
const route = new mapx.RouteControl({
  profile: "driving-truck",                  // Startprofil; Standard: erstes von `profiles`
  profiles: ["driving", "driving-truck", "cycling", "cycling-mtb", "walking"],
  onRoute: (response, request) => console.log(request.profile, response.routes[0].distance),
});
map.addControl(route, "top-right");
await route.setWaypoints([16.3725, 48.2085], [15.3294, 48.2272]); // beide Punkte aus dem Code setzen
```

Für Pkw bietet das Panel *Maut meiden* und *Autobahn meiden* (`exclude`), für Lkw Gewicht, Höhe, Breite und Länge. `route.last` enthält die letzte Antwort. [Live-Beispiel](/de/examples/sdk-controls/).

## LayerControl

Schaltet ganze Ebenengruppen ein und aus. Die Gruppen kommen aus den Style-Metadaten (`mapx:groups`); der Control funktioniert daher mit jedem mapx-Stil und zeigt neue Gruppen ohne Codeänderung. Overlays aus `mapx:overlays` (etwa Verkehr) werden ebenfalls angeboten, solange `groups` nicht eingeschränkt ist.

```js
map.addControl(new mapx.LayerControl({
  groups: ["buildings", "parcels", "relief", "bike_routes"],
  labels: { bike_routes: "Radrouten", relief: "Schummerung und Höhenlinien" },
}), "top-right");
```

## Suche, Stile, Gelände, Messen

```js
map.addControl(new mapx.SearchControl({ placeholder: "Suchen" }), "top-left");        // /v1/geocode
map.addControl(new mapx.StyleSwitcherControl({ styles: ["streets", "satellite", "outdoors"] }), "top-right");
map.addControl(new mapx.Terrain3DControl(), "top-right");
map.addControl(new mapx.MeasureControl(), "top-right");                                 // Länge und Fläche
map.addControl(new mapx.ElevationProfileControl(), "top-right");                        // Linie zeichnen, Profil erhalten
```

## ParcelPopup

Ein Klick zeigt das Grundstück an dieser Stelle: Nummer, Fläche, Katastralgemeinde und mit `dossier: true` (Standard) Gebäude und Höhenbereich. Der Umriss wird hervorgehoben.

```js
new mapx.ParcelPopup(map, { dossier: true, onParcel: (p) => console.log(p.properties.parcel_id) });
```

Ist das Dossier nicht verfügbar, fällt das Popup auf die Grunddaten des Grundstücks zurück.

## DeparturesPopup

Ein Klick sucht die nächste Haltestelle im `radius` (Meter) und listet die nächsten Abfahrten des Soll-Fahrplans.

```js
const departures = new mapx.DeparturesPopup(map, { limit: 8, radius: 150 });
departures.open("at:49:1320"); // oder per Haltestellen-ID, oder [lng, lat]
```

[Live-Beispiel](/de/examples/departures/).

## Popups und Werkzeuge kombinieren

Beide Popups reagieren auf jeden Kartenklick. Zusammen mit `RouteControl`, `MeasureControl` oder einem anderen Klick-Werkzeug die Popups bei Bedarf anlegen und mit `remove()` entfernen, solange das Werkzeug aktiv ist:

```js
let parcels = new mapx.ParcelPopup(map);
document.querySelector("#tools").onchange = (e) => (e.target.checked ? parcels.remove() : (parcels = new mapx.ParcelPopup(map)));
```

## Weiter

- [SDK-Helfer](/de/guides/sdk/helpers/)
- [Fehler und Wiederholungen](/de/guides/sdk/errors/)
- [SDK-Referenz](/reference/sdk/)
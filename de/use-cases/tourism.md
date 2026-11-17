# Tourismus: Radrouten und Höhenprofil

Source: https://docs.mapx.at/de/use-cases/tourism/

> Rad- und MTB-Routen, Radrouten-Ebene, Höhenprofil und Wetter für Touren und Gästeinformation: Anwendungsfall Tourismus mit den mapx-APIs.

Gäste fragen „Welchen Weg mit dem Rad, wie steil ist er, und wie wird das Wetter?“. Die Antworten kommen aus derselben API: Routen mit den Profilen `cycling` und `cycling-mtb`, ein Höhenprofil, die beschilderten Radrouten auf der Karte und die Vorhersage für das Ziel.

Alle Aufrufe brauchen einen Token; `DEIN_TOKEN` durch den eigenen ersetzen ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Die SDK-Beispiele setzen ihn einmal mit `mapx.setAccessToken("DEIN_TOKEN")`.

## Routen für Räder

`cycling` bevorzugt Radwege und beschilderte Routen und bewertet unbefestigte Abschnitte und starke Steigungen schlechter. `cycling-mtb` bevorzugt Forststraßen und Singletrails. Mit `steps=true` trägt jeder Schritt die Art der Infrastruktur und den Belag, `routes[].summary` enthält Anstieg, Abstieg und maximale Steigung.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/cycling-mtb/15.3294,48.2272;15.4,48.3?steps=true&overview=false" \
  | jq '.routes[0] | {distance, duration, summary}'
```

```js
const r = await mapx.directions("cycling", [[15.3294, 48.2272], [15.6141, 48.4097]], { steps: true, alternatives: true, annotations: ["elevation"] });
console.log(r.routes.map((x) => `${(x.distance / 1000).toFixed(1)} km, Anstieg ${x.summary?.ascent_m} m`));
```

## Radrouten anzeigen

Der Stil `streets` enthält die beschilderten Rad- und MTB-Routen als Ebenengruppe `bike_routes`. Sie lässt sich über die Style-Metadaten einschalten oder vom Nutzer umschalten:

```js
const map = mapx.map({ container: "map", style: "streets", center: [15.45, 48.3], zoom: 10 });
map.addControl(new mapx.LayerControl({ groups: ["bike_routes", "relief"] }), "top-right");
map.addControl(new mapx.RouteControl({ profiles: ["cycling", "cycling-mtb", "walking"] }), "top-right");
```

Das [SDK-Beispiel](https://docs.mapx.at/de/examples/sdk-controls.md) zeigt beide Controls zusammen.

## Höhenprofil

Jede Tourenseite gewinnt mit einem Profil. Die Routengeometrie in die Höhen-API geben (2 bis 512 Punkte; die vereinfachte Standardgeometrie der Route passt):

```js
const route = (await (await fetch("https://api.mapx.at/v1/directions/cycling/15.3294,48.2272;15.6141,48.4097", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).routes[0];
// overview=simplified (Standard) hält die Linie unter dem Limit von 512 Punkten
const p = await mapx.elevationProfile(route.geometry.coordinates, 120);
console.log(`Anstieg ${Math.round(p.ascent)} m, Abstieg ${Math.round(p.descent)} m, ${Math.round(p.min)}-${Math.round(p.max)} m`);
// p.distances[i] (m ab Start) und p.elevations[i] (m) ergeben das Diagramm
```

Das Höhenmodell ist das amtliche DGM des BEV. Ein fertiges Diagramm zeigt [Höhenprofil](https://docs.mapx.at/de/guides/elevation-profile.md), dazu das [Live-Beispiel](https://docs.mapx.at/de/examples/elevation-profile.md).

## Erreichbarkeit für die Region

Welche Ziele erreichen Gäste in 30 Minuten mit dem Rad? Eine Isochronen-Anfrage:

```js
const iso = await mapx.isochrone("cycling", [15.3294, 48.2272], { minutes: [15, 30, 45] }); // größte Fläche zuerst
map.addSource("reach", { type: "geojson", data: iso });
map.addLayer({ id: "reach", type: "fill", source: "reach", paint: { "fill-color": "#2346d8", "fill-opacity": 0.12 } });
```

Für den öffentlichen Verkehr zum Ausgangspunkt der Tour [`transitDirections`](https://docs.mapx.at/de/guides/sdk/helpers.md) mit `bike: true` nutzen, um mit dem Rad zur Haltestelle zu fahren.

## Wetter am Ziel

```js
const f = await mapx.weatherForecast([12.694, 47.074], { hours: 12 }); // Großglockner
console.log(f.hours.map((h) => `${h.time.slice(11, 16)} ${h.temperature_c}°C ${h.precipitation_mm} mm`).join("\n"));
```

`weatherCurrent` liefert die Messwerte der nächsten Stationen. Als Kartenebene siehe das [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md). Wetterdaten: GeoSphere Austria (CC BY 4.0).

## Nächste Schritte

- [Navigation](https://docs.mapx.at/de/guides/navigation.md) und [Höhe und Gelände](https://docs.mapx.at/de/guides/elevation.md)
- [3D-Gelände](https://docs.mapx.at/de/guides/3d-terrain.md) für eine eindrucksvolle Tourenkarte
- [Statische Bilder](https://docs.mapx.at/de/guides/static-images.md) für Prospekte und Newsletter

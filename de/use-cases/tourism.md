# Tourismus: Radrouten und Höhenprofil

Source: https://docs.mapx.at/de/use-cases/tourism/

> Rad- und MTB-Routen, Radrouten-Ebene, Höhenprofil und Wetter für Touren und Gästeinformation: Anwendungsfall Tourismus mit den mapx-APIs.

Gäste fragen „Welchen Weg mit dem Rad, wie steil ist er, und wie wird das Wetter?“. Die Antworten kommen aus derselben API: Routen mit den Profilen `cycling` und `cycling-mtb`, ein Höhenprofil, die beschilderten Radrouten auf der Karte und die Vorhersage für das Ziel.

Alle Aufrufe brauchen einen Token; `DEIN_TOKEN` durch den eigenen ersetzen ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)).

## Routen für Räder

`cycling` bevorzugt Radwege und beschilderte Routen und bewertet unbefestigte Abschnitte und starke Steigungen schlechter. `cycling-mtb` bevorzugt Forststraßen und Singletrails. Mit `steps=true` trägt jeder Schritt die Art der Infrastruktur und den Belag, `routes[].summary` enthält Anstieg, Abstieg und maximale Steigung.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/cycling-mtb/15.3294,48.2272;15.4,48.3?steps=true&overview=false" \
  | jq '.routes[0] | {distance, duration, summary}'
```

```js
const r = await (await fetch("https://api.mapx.at/v1/directions/cycling/15.3294,48.2272;15.6141,48.4097?steps=true&alternatives=true&annotations=elevation", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(r.routes.map((x) => `${(x.distance / 1000).toFixed(1)} km, Anstieg ${x.elevation_summary?.ascent} m`));
```

## Radrouten anzeigen

Der Stil `streets` enthält die beschilderten Rad- und MTB-Routen als Ebenengruppe `bike_routes`. Sie lässt sich über die Stil-Metadaten einschalten oder vom Nutzer umschalten:

```js
const TOKEN = "DEIN_TOKEN";
// Stil mit PMTiles-Quellen: pmtiles.js einbinden wie in den Beispielen
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map", style: "https://api.mapx.at/v1/styles/streets", center: [15.45, 48.3], zoom: 10,
  transformRequest: (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url }),
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
map.on("load", () => {
  // Ebenen der Gruppe bike_routes einschalten (im Stil standardmäßig aus)
  for (const id of map.getStyle().metadata["mapx:groups"].bike_routes) map.setLayoutProperty(id, "visibility", "visible");
});
```

Eine Route auf der Karte berechnen und zeichnen zeigt das Beispiel [Route](https://docs.mapx.at/de/examples/route.md) (MapLibre, OpenLayers, Leaflet).

## Höhenprofil

Jede Tourenseite gewinnt mit einem Profil. Die Routengeometrie in die Höhen-API geben (2 bis 512 Punkte; die vereinfachte Standardgeometrie der Route passt):

```js
const route = (await (await fetch("https://api.mapx.at/v1/directions/cycling/15.3294,48.2272;15.6141,48.4097", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).routes[0];
// overview=simplified (Standard) hält die Linie unter dem Limit von 512 Punkten
const line = route.geometry.coordinates.map((c) => c.join(",")).join(";");
const p = (await (await fetch(`https://api.mapx.at/v1/elevation/profile?coordinates=${line}&samples=120`, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).properties;
console.log(`Anstieg ${Math.round(p.ascent)} m, Abstieg ${Math.round(p.descent)} m, ${Math.round(p.min_elevation)}-${Math.round(p.max_elevation)} m`);
// p.distances[i] (m ab Start) und p.elevations[i] (m) ergeben das Diagramm
```

Das Höhenmodell ist das DTM des BEV. Ein fertiges Diagramm zeigt [Höhenprofil](https://docs.mapx.at/de/guides/elevation-profile.md), dazu das [Live-Beispiel](https://docs.mapx.at/de/examples/elevation-profile.md).

## Erreichbarkeit für die Region

Welche Ziele erreichen Gäste in 30 Minuten mit dem Rad? Eine Isochronen-Anfrage:

```js
const iso = await (await fetch("https://api.mapx.at/v1/isochrone/cycling/15.3294,48.2272?contours_minutes=15,30,45", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json(); // größte Fläche zuerst
map.addSource("reach", { type: "geojson", data: iso });
map.addLayer({ id: "reach", type: "fill", source: "reach", paint: { "fill-color": "#2346d8", "fill-opacity": 0.12 } });
```

Für den öffentlichen Verkehr zum Ausgangspunkt der Tour das Profil `bike_transit` nutzen (`GET /v1/directions/bike_transit/{start};{ziel}`, [ÖV](https://docs.mapx.at/de/guides/transit.md)), um mit dem Rad zur Haltestelle zu fahren.

## Wetter am Ziel

```js
const f = await (await fetch("https://api.mapx.at/v1/weather/forecast?lng=12.694&lat=47.074&hours=12", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json(); // Großglockner
console.log(f.hours.map((h) => `${new Date(h.time).toLocaleTimeString("de-AT", { hour: "2-digit", minute: "2-digit", timeZone: "Europe/Vienna" })} ${h.temperature_c}°C ${h.precipitation_mm} mm`).join("\n"));
```

`GET /v1/weather/current?lng=…&lat=…` liefert die Messwerte der nächsten Stationen. Als Kartenebene siehe das [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md). Wetterdaten: GeoSphere Austria (CC BY 4.0).

## Nächste Schritte

- [Navigation](https://docs.mapx.at/de/guides/navigation.md) und [Höhe und Gelände](https://docs.mapx.at/de/guides/elevation.md)
- [3D-Gelände](https://docs.mapx.at/de/guides/3d-terrain.md) für eine eindrucksvolle Tourenkarte
- [Statische Bilder](https://docs.mapx.at/de/guides/static-images.md) für Prospekte und Newsletter

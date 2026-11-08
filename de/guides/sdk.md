# SDK mapx-gl

Ein kleines Add-on für MapLibre GL JS: Karten-Factory, Suche, Style-Umschalter, Messen, Höhenprofil, 3D-Gelände und API-Helfer.

`mapx-gl` ist ein kleines Add-on (ESM, TypeScript) für MapLibre GL JS. Es setzt die Style-URLs, registriert das `pmtiles://`-Protokoll und ergänzt fertige Controls und Helfer für die [APIs](/de/api/). Peer-Abhängigkeiten: `maplibre-gl@{V.maplibre}` und `pmtiles@{V.pmtiles}`.

## Installation

Die Dateien von `https://map.mapx.at/sdk/v1/` herunterladen und zusammen mit der Seite ausliefern:

| Datei | Verwendung |
|---|---|
| `mapx-gl.bundle.js` | Alles enthalten, als `<script type="module">` laden |
| `mapx-gl.js` | Ohne Peers, für Projekte mit eigenem Build |
| `mapx-gl.css` | Styles für die Controls, nach `maplibre-gl.css` laden |

```html
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
<link rel="stylesheet" href="./mapx-gl.css">
<div id="map" style="height:100vh"></div>
<script type="module">
  import mapx from "./mapx-gl.bundle.js";
  const map = mapx.map({ container: "map", style: "streets", center: [15.33, 48.23], zoom: 12 });
  map.addControl(new mapx.SearchControl(), "top-left");
  map.addControl(new mapx.StyleSwitcherControl(), "top-right");
</script>
```

Ein vollständiges Beispiel liegt unter [map.mapx.at/sdk/v1/example.html](https://map.mapx.at/sdk/v1/example.html).

## Karte

`mapx.map(options)` liefert eine MapLibre-Karte. `style` ist ein Kurzname (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`), der zu `https://api.mapx.at/v1/styles/{id}` wird, oder eine beliebige Style-URL bzw. ein Style-Objekt. `terrain: true` schaltet 3D-Gelände ein.

## Controls

| Control | Funktion | API |
|---|---|---|
| `SearchControl` | Suchbox mit Ergebnisliste | [Geocoding](/de/api/geocoding/) |
| `StyleSwitcherControl` | Zwischen Styles wechseln | [Styles](/de/api/styles/) |
| `MeasureControl` | Distanzen messen | keine |
| `ElevationProfileControl` | Höhenprofil einer gezeichneten Linie | [Elevation](/de/api/elevation/) |
| `Terrain3DControl` | 3D-Gelände ein und aus | [Tiles](/de/api/tiles/) |

## Helfer

| Helfer | Liefert |
|---|---|
| `mapx.geocode(q, { limit, types, bbox, proximity })` | Liste von Features |
| `mapx.directions(profile, [[lng, lat], ...], { steps, overview })` | Directions-Antwort |
| `mapx.isochrone(profile, [lng, lat], { minutes })` | GeoJSON-Polygone |
| `mapx.elevation(point or points, model)` | Höhe oder Liste von Höhen in m |
| `mapx.elevationProfile(line, samples, model)` | Distanzen, Höhen, Anstieg, Abstieg |
| `mapx.setApiBase(url)` | Anderen API-Host verwenden, zum Beispiel für Tests |

```js
const r = await mapx.directions("walking", [[15.33, 48.22], [15.34, 48.23]]);
const h = await mapx.elevation([12.694, 47.074]); // 3726.3
```

## Nächste Schritte

- [Beispiele](/de/examples/)
- [Karten](/de/guides/maps/) und [Navigation](/de/guides/navigation/)
- [API-Referenz](/de/api/)
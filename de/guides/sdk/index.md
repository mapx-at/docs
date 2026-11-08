# SDK mapx-gl


`mapx-gl` ist ein kleines Add-on (ESM, TypeScript) für MapLibre GL JS. Es setzt die Style-URLs, registriert das `pmtiles://`-Protokoll, verpackt die [APIs](/de/api/) in typisierte Funktionen und bringt fertige Controls mit. Peer-Abhängigkeiten: `maplibre-gl@{V.maplibre}` und `pmtiles@{V.pmtiles}`.

## Installation

Das Bundle von `https://map.mapx.at/sdk/v1/` laden (CORS ist offen) oder die Dateien herunterladen und mit der eigenen Seite ausliefern:

| Datei | Verwendung |
|---|---|
| `mapx-gl.bundle.js` | Alles enthalten, als `<script type="module">` laden |
| `mapx-gl.js` | Ohne Peers, für Projekte mit eigenem Build |
| `mapx-gl.css` | Stile der Controls, nach `maplibre-gl.css` laden |
| `mapx-gl.d.ts` | TypeScript-Typen |

```html
<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
<link rel="stylesheet" href="https://map.mapx.at/sdk/v1/mapx-gl.css">
<div id="map" style="height:100vh"></div>
<script type="module">
  import mapx from "https://map.mapx.at/sdk/v1/mapx-gl.bundle.js";
  const map = mapx.map({ accessToken: "DEIN_TOKEN", container: "map", style: "streets", center: [15.33, 48.23], zoom: 12 });
  map.addControl(new mapx.SearchControl(), "top-left");
  map.addControl(new mapx.RouteControl(), "top-right");
  map.addControl(new mapx.LayerControl(), "top-right");
</script>
```

Ein vollständiges Beispiel liegt unter [map.mapx.at/sdk/v1/example.html](https://map.mapx.at/sdk/v1/example.html).

## Token

Alle Aufrufe brauchen einen Token ([Zugang und Limits](/de/start/access-and-limits/)). Er wird einmal gesetzt, mit `mapx.map({ accessToken: "DEIN_TOKEN", … })` oder `mapx.setAccessToken("DEIN_TOKEN")`; Helfer, Styles, Tiles und `staticImageUrl()` tragen ihn dann mit. Auf Webseiten einen öffentlichen Token (`mx_pk_…`) verwenden, geheime Tokens (`mx_sk_…`) gehören nur auf Server.

## Karte

`mapx.map(options)` liefert eine MapLibre-Karte. `style` ist ein Kurzname (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`), der zu `https://api.mapx.at/v1/styles/{id}` wird, oder eine beliebige Style-URL bzw. ein Style-Objekt. `terrain: true` schaltet das 3D-Gelände ein.

## Was dabei ist

| Teil | Inhalt | Seite |
|---|---|---|
| Helfer | `geocode`, `reverse`, `searchSuggest`, `searchRetrieve`, `directions`, `transitDirections`, `isochrone`, `matrix`, `optimization`, `parcel`, `parcelDossier`, `tilequery`, `elevation`, `elevationProfile`, `trafficIncidents`, `stops`, `departures`, `weatherCurrent`, `weatherForecast`, `staticImageUrl` | [SDK-Helfer](/de/guides/sdk/helpers/) |
| Controls | `SearchControl`, `StyleSwitcherControl`, `MeasureControl`, `ElevationProfileControl`, `Terrain3DControl`, `RouteControl`, `LayerControl` | [Controls und Popups](/de/guides/sdk/controls/) |
| Popups | `ParcelPopup`, `DeparturesPopup` | [Controls und Popups](/de/guides/sdk/controls/) |
| Fehler | `MapxError` mit `code`, `param`, `allowed`, `retryAfter` | [Fehler und Wiederholungen](/de/guides/sdk/errors/) |
| Typen | Alle Options- und Antworttypen | [SDK-Referenz](/reference/sdk/) |

```js
const r = await mapx.directions("driving-truck", [[16.3725, 48.2085], [15.3294, 48.2272]], { truck: { weight: 40, height: 4 } });
const h = await mapx.elevation([12.694, 47.074]); // 3726.3
```

## Weiter

- [Beispiele](/de/examples/) mit SDK: [Routenplaner und Ebenen](/de/examples/sdk-controls/), [Abfahrten](/de/examples/departures/)
- [Karten](/de/guides/maps/) und [Navigation](/de/guides/navigation/)
- [API-Referenz](/de/api/)


# SDK mapx-gl

Kleines MapLibre-SDK für die mapx-APIs: Karte, Suche, Stilwechsel, Messen, Höhenprofil, 3D-Gelände.

`mapx-gl` ist ein kleines Aufsatz-SDK (ESM, TypeScript) für MapLibre GL JS. Es setzt die Stil-URLs, registriert das `pmtiles://`-Protokoll und bringt fertige Bedienelemente und Helfer für die [APIs](/api/) mit. Peer-Abhängigkeiten: `maplibre-gl@4.7.1`, `pmtiles@3.2.1`.

Auslieferung: [https://map.mapx.at/sdk/v1/](https://map.mapx.at/sdk/v1/) (`mapx-gl.bundle.js` mit allem, `mapx-gl.js` mit externen Peers, `mapx-gl.css`, `example.html`).

<Code lang="js" code={`import mapx from 'mapx-gl'; // oder https://map.mapx.at/sdk/v1/mapx-gl.bundle.js (alles enthalten)

const map = mapx.map({ container: 'map', style: 'streets', center: [15.33, 48.23], zoom: 12, terrain: false });
map.addControl(new mapx.SearchControl(), 'top-left');
map.addControl(new mapx.StyleSwitcherControl(), 'top-right');`} />

## Karte

`mapx.map(opts)` liefert eine MapLibre-Karte. `style` ist ein Kurzname (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`) und wird zu `https://api.mapx.at/v1/styles/{id}` ([Styles](/api/styles/)). Das SDK registriert `pmtiles://` und schaltet mit `terrain` optional das 3D-Gelände ein.

## Bedienelemente

| Element | Funktion | API |
|---|---|---|
| `SearchControl` | Suchfeld mit Trefferliste | [`/v1/geocode`](/api/geocoding/) |
| `StyleSwitcherControl` | zwischen Stilen umschalten | [Styles](/api/styles/) |
| `MeasureControl` | Strecken messen | lokal |
| `ElevationProfileControl` | Höhenprofil einer Linie | [`/v1/elevation/profile`](/api/hoehe/#profil-endpunkt) |
| `Terrain3DControl` | 3D-Gelände ein und aus | [TileJSON](/api/tilejson/) |

## Helfer

- `directions(profile, [[lng,lat],...])` – Route ([Routing](/api/routing/#directions))
- `isochrone(profile, [lng,lat], {minutes})` – Erreichbarkeit ([Routing](/api/routing/#isochrone))
- `elevation(point|points)` und `elevationProfile(line)` – [Höhe und Profil](/api/hoehe/)
- `geocode(q)` – [Suche](/api/geocoding/)
- `setApiBase(url)` – anderen API-Host verwenden (zum Beispiel zum Testen)

## CSS

`mapx-gl.css` zusätzlich zu `maplibre-gl.css` einbinden:

<Code lang="html" code={`<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
<link rel="stylesheet" href="https://map.mapx.at/sdk/v1/mapx-gl.css">`} />

## Bundle ohne Build

<Code lang="html" code={`<script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
<script src="https://unpkg.com/pmtiles@3.2.1/dist/pmtiles.js"></script>
<script src="https://map.mapx.at/sdk/v1/mapx-gl.js"></script>`} />

Das eigenständige `mapx-gl.bundle.js` enthält die Peers bereits; die Variante ohne Peers (`mapx-gl.js`) ist für Projekte mit eigenem Build gedacht. Ein vollständiges Beispiel liegt unter [https://map.mapx.at/sdk/v1/example.html](https://map.mapx.at/sdk/v1/example.html).
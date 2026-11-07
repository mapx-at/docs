# SDK mapx-gl

Small MapLibre SDK for the mapx APIs: map, search, style switching, measuring, elevation profile, 3D terrain.

`mapx-gl` is a small add-on SDK (ESM, TypeScript) for MapLibre GL JS. It sets the style URLs, registers the `pmtiles://` protocol and ships ready-made controls and helpers for the [APIs](/en/api/). Peer dependencies: `maplibre-gl@4.7.1`, `pmtiles@3.2.1`.

Delivery: [https://map.mapx.at/sdk/v1/](https://map.mapx.at/sdk/v1/) (`mapx-gl.bundle.js` with everything, `mapx-gl.js` with external peers, `mapx-gl.css`, `example.html`).

<Code lang="js" code={`import mapx from 'mapx-gl'; // or https://map.mapx.at/sdk/v1/mapx-gl.bundle.js (everything included)

const map = mapx.map({ container: 'map', style: 'streets', center: [15.33, 48.23], zoom: 12, terrain: false });
map.addControl(new mapx.SearchControl(), 'top-left');
map.addControl(new mapx.StyleSwitcherControl(), 'top-right');`} />

## Map

`mapx.map(opts)` returns a MapLibre map. `style` is a short name (`streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `outdoors`) and becomes `https://api.mapx.at/v1/styles/{id}` ([Styles](/en/api/styles/)). The SDK registers `pmtiles://` and optionally enables 3D terrain with `terrain`.

## Controls

| Control | Function | API |
|---|---|---|
| `SearchControl` | search box with result list | [`/v1/geocode`](/en/api/geocoding/) |
| `StyleSwitcherControl` | switch between styles | [Styles](/en/api/styles/) |
| `MeasureControl` | measure distances | local |
| `ElevationProfileControl` | elevation profile of a line | [`/v1/elevation/profile`](/en/api/hoehe/#profile-endpoint) |
| `Terrain3DControl` | 3D terrain on and off | [TileJSON](/en/api/tilejson/) |

## Helpers

- `directions(profile, [[lng,lat],...])` – route ([Routing](/en/api/routing/#directions))
- `isochrone(profile, [lng,lat], {minutes})` – reachability ([Routing](/en/api/routing/#isochrone))
- `elevation(point|points)` and `elevationProfile(line)` – [Elevation and profile](/en/api/hoehe/)
- `geocode(q)` – [Search](/en/api/geocoding/)
- `setApiBase(url)` – use a different API host (for example for testing)

## CSS

Include `mapx-gl.css` in addition to `maplibre-gl.css`:

<Code lang="html" code={`<link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
<link rel="stylesheet" href="https://map.mapx.at/sdk/v1/mapx-gl.css">`} />

## Bundle without a build

<Code lang="html" code={`<script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
<script src="https://unpkg.com/pmtiles@3.2.1/dist/pmtiles.js"></script>
<script src="https://map.mapx.at/sdk/v1/mapx-gl.js"></script>`} />

The standalone `mapx-gl.bundle.js` already contains the peers; the variant without peers (`mapx-gl.js`) is meant for projects with their own build. A complete example is at [https://map.mapx.at/sdk/v1/example.html](https://map.mapx.at/sdk/v1/example.html).
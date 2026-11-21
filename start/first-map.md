# First map with MapLibre in 5 minutes

Source: https://docs.mapx.at/start/first-map/

> Build your first mapx web map: a complete HTML file with MapLibre GL JS and a public token. Copy, save and open it in a browser.

Save the code as `index.html`, run `npx serve` in the folder and open the printed address (`http://localhost:3000`). Replace `YOUR_TOKEN` with a public token from [account.mapx.at](https://docs.mapx.at/start/access-and-limits.md) and add `http://localhost:3000` as an allowed origin for the token. Double-clicking the file is not enough: without a web server there is no origin and the API answers `403`. No build step.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>First mapx map</title>
  <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.css">
  <script src="https://unpkg.com/pmtiles@4.5.0/dist/pmtiles.js"></script>
  <style>html, body, #map { margin: 0; height: 100%; }</style>
</head>
<body>
  <div id="map"></div>
  <script type="module">
    import * as maplibregl from "https://unpkg.com/maplibre-gl@6.19.0/dist/maplibre-gl.mjs";
    const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
    // the style loads PMTiles files: register the pmtiles:// protocol once
    maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
    const map = new maplibregl.Map({
      attributionControl: false,
      container: "map",
      style: "https://api.mapx.at/v1/styles/streets",
      // adds the token to every request to mapx.at (style, tiles, fonts, sprites)
      transformRequest: (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url }),
      center: [16.3725, 48.2085], // Vienna, lon,lat
      zoom: 13,
    });
    map.addControl(new maplibregl.AttributionControl({ compact: true }));
    map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // start collapsed (MapLibre keeps it open until the first drag)
  </script>
</body>
</html>
```

**Prerequisite:** the mapx styles load their tiles from PMTiles files. The page must therefore include `pmtiles.js` and call `maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile)` once; otherwise the map stays empty (console: `URL scheme "pmtiles" is not supported`). Your own styles that only use TileJSON sources (`https://tiles.mapx.at/v1/tiles/{tileset}.json`) do not need it.

The style `streets` is a normal [MapLibre style](https://maplibre.org/maplibre-style-spec/). Replace it with `satellite` or `outdoors` to change the look. All styles: [Styles API](https://docs.mapx.at/api/styles.md).

MapLibre GL JS 6.19.0 is the version these examples are tested with.

## Next

- [First search](https://docs.mapx.at/start/first-search.md)
- [Maps guide](https://docs.mapx.at/guides/maps.md): styles, terrain, overlays
- [Examples](https://docs.mapx.at/examples/index.md)

# First map with MapLibre in 5 minutes

Source: https://docs.mapx.at/start/first-map/

> Build your first mapx web map: a complete HTML file with MapLibre GL JS or the mapx-gl SDK and a public token. Copy, save and open it in a browser.

Save the code as `index.html` and open it in a browser. Replace `YOUR_TOKEN` with a public token from [account.mapx.at](https://docs.mapx.at/start/access-and-limits.md) (allowed origin: your site, or `localhost` while testing). No build step.

**MapLibre GL JS**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>First mapx map</title>
  <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.css">
  <style>html, body, #map { margin: 0; height: 100%; }</style>
</head>
<body>
  <div id="map"></div>
  <script type="module">
    import * as maplibregl from "https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.mjs";
    const TOKEN = "YOUR_TOKEN"; // public token mx_pk_…
    new maplibregl.Map({
      container: "map",
      style: "https://api.mapx.at/v1/styles/streets",
      // adds the token to every request to mapx.at (style, tiles, fonts, sprites)
      transformRequest: (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url }),
      center: [16.3725, 48.2085], // Vienna, lon,lat
      zoom: 13,
    });
  </script>
</body>
</html>
```

**SDK mapx-gl**

The [SDK](https://docs.mapx.at/guides/sdk/index.md) adds a search box and a style switcher. Download `mapx-gl.bundle.js`, `maplibre-gl-worker.js` and `mapx-gl.css` from `https://map.mapx.at/sdk/v1/` and serve them next to your page (the worker must sit in the same directory as the bundle).

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>First mapx map</title>
  <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@6.18.0/dist/maplibre-gl.css">
  <link rel="stylesheet" href="./mapx-gl.css">
  <style>html, body, #map { margin: 0; height: 100%; }</style>
</head>
<body>
  <div id="map"></div>
  <script type="module">
    import mapx from "./mapx-gl.bundle.js";
    const map = mapx.map({ accessToken: "YOUR_TOKEN", container: "map", style: "streets", center: [16.3725, 48.2085], zoom: 13 });
    map.addControl(new mapx.SearchControl(), "top-left");
    map.addControl(new mapx.StyleSwitcherControl(), "top-right");
  </script>
</body>
</html>
```

The style `streets` is a normal [MapLibre style](https://maplibre.org/maplibre-style-spec/). Replace it with `satellite` or `outdoors` to change the look. All styles: [Styles API](https://docs.mapx.at/api/styles.md).

MapLibre GL JS 6.18.0 is the version these examples are tested with.

## Next

- [First search](https://docs.mapx.at/start/first-search.md)
- [Maps guide](https://docs.mapx.at/guides/maps.md): styles, terrain, overlays
- [Examples](https://docs.mapx.at/examples/index.md)

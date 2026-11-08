# First map in 5 minutes

A complete HTML file that shows a mapx map. Copy, save, open.

Save the code as `index.html` and open it in a browser. Replace `YOUR_TOKEN` with a public token from [account.mapx.at](/start/access-and-limits/) (allowed origin: your site, or `localhost` while testing). No build step.

<Tabs syncKey="client">
<TabItem label="MapLibre GL JS">

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>First mapx map</title>
  <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
  <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>
  <style>html, body, #map { margin: 0; height: 100%; }</style>
</head>
<body>
  <div id="map"></div>
  <script>
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

</TabItem>
<TabItem label="SDK mapx-gl">

The [SDK](/guides/sdk/) adds a search box and a style switcher. Download `mapx-gl.bundle.js` and `mapx-gl.css` from `https://map.mapx.at/sdk/v1/` and serve them next to your page.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>First mapx map</title>
  <link rel="stylesheet" href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css">
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

</TabItem>
</Tabs>

The style `streets` is a normal [MapLibre style](https://maplibre.org/maplibre-style-spec/). Replace it with `satellite` or `outdoors` to change the look. All styles: [Styles API](/api/styles/).

MapLibre GL JS {V.maplibre} is the version these examples are tested with.

## Next

- [First search](/start/first-search/)
- [Maps guide](/guides/maps/): styles, terrain, overlays
- [Examples](/examples/)
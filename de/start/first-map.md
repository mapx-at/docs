# Erste Karte mit MapLibre in 5 Minuten

Source: https://docs.mapx.at/de/start/first-map/

> Die erste mapx-Webkarte: eine vollständige HTML-Datei mit MapLibre GL JS oder dem SDK mapx-gl und einem öffentlichen Token. Kopieren, speichern, im Browser öffnen.

Den Code als `index.html` speichern und im Browser öffnen. `DEIN_TOKEN` durch einen öffentlichen Token von [account.mapx.at](https://docs.mapx.at/de/start/access-and-limits.md) ersetzen (erlaubter Origin: die eigene Seite, beim Testen `localhost`). Kein Build-Schritt.

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
    const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
    new maplibregl.Map({
      container: "map",
      style: "https://api.mapx.at/v1/styles/streets",
      // hängt den Token an jede Anfrage an mapx.at (Style, Tiles, Schriften, Sprites)
      transformRequest: (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url }),
      center: [16.3725, 48.2085], // Wien, lon,lat
      zoom: 13,
    });
  </script>
</body>
</html>
```

**SDK mapx-gl**

Das [SDK](https://docs.mapx.at/de/guides/sdk/index.md) ergänzt eine Suchbox und einen Style-Umschalter. `mapx-gl.bundle.js`, `maplibre-gl-worker.js` und `mapx-gl.css` von `https://map.mapx.at/sdk/v1/` herunterladen und neben der Seite ausliefern (der Worker muss im selben Verzeichnis wie das Bundle liegen).

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
    const map = mapx.map({ accessToken: "DEIN_TOKEN", container: "map", style: "streets", center: [16.3725, 48.2085], zoom: 13 });
    map.addControl(new mapx.SearchControl(), "top-left");
    map.addControl(new mapx.StyleSwitcherControl(), "top-right");
  </script>
</body>
</html>
```

Der Style `streets` ist ein gewöhnlicher [MapLibre-Style](https://maplibre.org/maplibre-style-spec/). Durch `satellite` oder `outdoors` ersetzen, um das Aussehen zu ändern. Alle Styles: [Styles API](https://docs.mapx.at/de/api/styles.md).

MapLibre GL JS 6.18.0 ist die Version, mit der diese Beispiele getestet sind.

## Nächste Schritte

- [Erste Suche](https://docs.mapx.at/de/start/first-search.md)
- [Karten-Guide](https://docs.mapx.at/de/guides/maps.md): Styles, Gelände, Overlays
- [Beispiele](https://docs.mapx.at/de/examples/index.md)

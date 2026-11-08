# Erste Karte in 5 Minuten

Eine vollständige HTML-Datei mit mapx-Karte. Kopieren, speichern, öffnen.

Den Code als `index.html` speichern und im Browser öffnen. Kein Key, kein Build-Schritt.

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
    new maplibregl.Map({
      container: "map",
      style: "https://api.mapx.at/v1/styles/streets",
      center: [16.3725, 48.2085], // Wien, lon,lat
      zoom: 13,
    });
  </script>
</body>
</html>
```

</TabItem>
<TabItem label="SDK mapx-gl">

Das [SDK](/de/guides/sdk/) ergänzt eine Suchbox und einen Style-Umschalter. `mapx-gl.bundle.js` und `mapx-gl.css` von `https://map.mapx.at/sdk/v1/` herunterladen und neben der Seite ausliefern.

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
    const map = mapx.map({ container: "map", style: "streets", center: [16.3725, 48.2085], zoom: 13 });
    map.addControl(new mapx.SearchControl(), "top-left");
    map.addControl(new mapx.StyleSwitcherControl(), "top-right");
  </script>
</body>
</html>
```

</TabItem>
</Tabs>

Der Style `streets` ist ein gewöhnlicher [MapLibre-Style](https://maplibre.org/maplibre-style-spec/). Durch `satellite` oder `outdoors` ersetzen, um das Aussehen zu ändern. Alle Styles: [Styles API](/de/api/styles/).

MapLibre GL JS {V.maplibre} ist die Version, mit der diese Beispiele getestet sind.

## Nächste Schritte

- [Erste Suche](/de/start/first-search/)
- [Karten-Guide](/de/guides/maps/): Styles, Gelände, Overlays
- [Beispiele](/de/examples/)
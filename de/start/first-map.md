# Erste Karte mit MapLibre in 5 Minuten

Source: https://docs.mapx.at/de/start/first-map/

> Die erste mapx-Webkarte: eine vollständige HTML-Datei mit MapLibre GL JS und einem öffentlichen Token. Kopieren, speichern, im Browser öffnen.

Den Code als `index.html` speichern und im Browser öffnen. `DEIN_TOKEN` durch einen öffentlichen Token von [account.mapx.at](https://docs.mapx.at/de/start/access-and-limits.md) ersetzen (erlaubter Origin: die eigene Seite, beim Testen `localhost`). Kein Build-Schritt.

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
    const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
    // der Stil lädt PMTiles-Dateien: Protokoll pmtiles:// registrieren
    maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
    const map = new maplibregl.Map({
      attributionControl: false,
      container: "map",
      style: "https://api.mapx.at/v1/styles/streets",
      // hängt den Token an jede Anfrage an mapx.at (Style, Tiles, Schriften, Sprites)
      transformRequest: (url) => ({ url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url }),
      center: [16.3725, 48.2085], // Wien, lon,lat
      zoom: 13,
    });
    map.addControl(new maplibregl.AttributionControl({ compact: true }));
    map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
  </script>
</body>
</html>
```

Der Style `streets` ist ein gewöhnlicher [MapLibre-Style](https://maplibre.org/maplibre-style-spec/). Durch `satellite` oder `outdoors` ersetzen, um das Aussehen zu ändern. Alle Styles: [Styles API](https://docs.mapx.at/de/api/styles.md).

MapLibre GL JS 6.19.0 ist die Version, mit der diese Beispiele getestet sind.

## Nächste Schritte

- [Erste Suche](https://docs.mapx.at/de/start/first-search.md)
- [Karten-Guide](https://docs.mapx.at/de/guides/maps.md): Styles, Gelände, Overlays
- [Beispiele](https://docs.mapx.at/de/examples/index.md)

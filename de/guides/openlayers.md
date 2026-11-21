# mapx-Karten mit OpenLayers

Source: https://docs.mapx.at/de/guides/openlayers/

> mapx-Basiskarte in OpenLayers einbinden: gerenderte Rasterkacheln des Stils, Token, Quellenangabe, eigenes GeoJSON und die Grenzen gegenüber MapLibre.

**Ziel:** Eine mapx-Karte in einer bestehenden OpenLayers-Anwendung (10.11.0), ohne Build, mit kostenlosem öffentlichem Token (siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)).

## Basiskarte einbinden

Die Stile von mapx sind MapLibre-Stile mit Vektorkacheln. OpenLayers bekommt denselben Stil fertig gerendert als Rasterkacheln ([Static Tiles](https://docs.mapx.at/de/api/styles.md)): `ol/source/XYZ` mit Kachelgröße 512. Stile: `streets` (und Varianten), `satellite`, `satellite-streets`, `outdoors`.

```html
<link rel="stylesheet" href="https://unpkg.com/ol@10.11.0/ol.css">
<script src="https://unpkg.com/ol@10.11.0/dist/ol.js"></script>
<div id="map" style="height:70vh"></div>
<script>
const TOKEN = 'DEIN_TOKEN';
const map = new ol.Map({
  target: 'map',
  layers: [new ol.layer.Tile({ source: new ol.source.XYZ({
    url: 'https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN,
    tileSize: 512, maxZoom: 20, attributionsCollapsible: false,
    attributions: '<a href="https://mapx.at">mapx.at</a>, <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
  }) })],
  view: new ol.View({ center: ol.proj.fromLonLat([16.37, 48.21]), zoom: 13 }),
});
</script>
```

Der Token steht als `access_token` in der Kachel-URL, API-Aufrufe per `fetch` schicken ihn als Header `Authorization: Bearer …`.

## Quellenangabe

Die Daten stehen unter CC BY 4.0, die Quelle muss auf der Karte erreichbar sein. Bei Rasterkacheln gibt es kein TileJSON, aus dem OpenLayers den Text liest: `attributions` selbst setzen. `attributionsCollapsible: false` zeigt den Text dauerhaft; ohne diese Option legt OpenLayers ihn hinter einen Knopf „i“, wie die MapLibre-Beispiele dieser Doku. Den Text je Quelle nennt [Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md).

## Eigenes GeoJSON

Antworten der APIs (Route, Isochrone …) sind GeoJSON in WGS84. `ol.format.GeoJSON` liest sie in die Kartenprojektion:

```js
const quelle = new ol.source.Vector();
map.addLayer(new ol.layer.Vector({ source: quelle }));
quelle.addFeatures(new ol.format.GeoJSON().readFeatures(geojson, { featureProjection: 'EPSG:3857' }));
```

## Grenzen

- Kein Vektorstil: die Basiskarte kommt als Bild, Beschriftungen und Ebenen lassen sich nicht einzeln ändern. Dafür [MapLibre](https://docs.mapx.at/de/guides/maps.md).
- Kein 3D-Gelände und keine 3D-Gebäude, die gibt es nur in MapLibre.
- Rasterkacheln zählen in der Limit-Zone `static` (5 Anfragen pro Sekunde je IP, kurzzeitig bis 60 auf einmal, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Gewöhnliches Verschieben und Zoomen bleibt deutlich darunter; erst wer dauerhaft schneller neue Kacheln anfordert, bekommt `429`.

## Beispiele

Die Beispiele mit OpenLayers-Fassung stehen in der [Übersicht](https://docs.mapx.at/de/examples/index.md) (Filter „OpenLayers“), etwa [Karte einbetten](https://docs.mapx.at/de/examples/embed-map.md#ol) und [Route](https://docs.mapx.at/de/examples/route.md#ol).

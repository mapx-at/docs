# mapx-Karten mit Leaflet

Source: https://docs.mapx.at/de/guides/leaflet/

> mapx-Basiskarte in Leaflet einbinden: gerenderte Rasterkacheln des Stils, Token, Quellenangabe, eigenes GeoJSON und die Grenzen gegenüber MapLibre.

**Ziel:** Eine mapx-Karte in einer bestehenden Leaflet-Anwendung (1.9.4), ohne Build und ohne Plugin, mit kostenlosem öffentlichem Token (siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)).

## Basiskarte einbinden

Leaflet zeichnet keine Vektorkacheln. Die mapx-Stile gibt es deshalb fertig gerendert als Rasterkacheln ([Static Tiles](https://docs.mapx.at/de/api/styles.md)), eingebunden mit `L.tileLayer`. Die Kacheln sind 512 Pixel groß: `tileSize: 512` und `zoomOffset: -1` halten den Maßstab wie bei gewöhnlichen 256er-Kacheln. Stile: `streets` (und Varianten), `light`, `dark`, `satellite`, `satellite-streets`, `outdoors`, `outdoors-winter`; nicht als Rasterkachel verfügbar sind `traffic-day`, `traffic-night`, `satellite-current`, die Reliefstile und die `-3d`-Stile.

```html
<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<div id="map" style="height:70vh"></div>
<script>
const TOKEN = 'DEIN_TOKEN';
const map = L.map('map').setView([48.21, 16.37], 13);
L.tileLayer('https://api.mapx.at/v1/styles/streets/tiles/512/{z}/{x}/{y}?access_token=' + TOKEN, {
  tileSize: 512, zoomOffset: -1, minZoom: 1, maxZoom: 21,
  attribution: '<a href="https://mapx.at">mapx.at</a> | <a href="https://www.bev.gv.at">BEV</a>, <a href="https://www.gip.gv.at">GIP.at</a>, <a href="https://creativecommons.org/licenses/by/4.0/">CC BY 4.0</a>',
}).addTo(map);
</script>
```

Der Token steht als `access_token` in der Kachel-URL, API-Aufrufe per `fetch` schicken ihn als Header `Authorization: Bearer …`.

## Quellenangabe

Die Daten stehen unter CC BY 4.0, die Quelle muss auf der Karte erreichbar sein. Leaflet zeigt den Text aus `attribution` unten rechts an; den Text je Quelle nennt [Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md).

## Eigenes GeoJSON

Antworten der APIs (Route, Isochrone …) sind GeoJSON und gehen direkt in `L.geoJSON`:

```js
const ebene = L.geoJSON(null, { style: { color: '#2346d8', weight: 4 } }).addTo(map);
ebene.addData(geojson);
```

## Grenzen

- Kein Vektorstil: die Basiskarte kommt als Bild, Beschriftungen und Ebenen lassen sich nicht einzeln ändern. Dafür [MapLibre](https://docs.mapx.at/de/guides/maps.md).
- Kein 3D-Gelände und keine 3D-Gebäude, die gibt es nur in MapLibre.
- Rasterkacheln zählen in der Limit-Zone `static` (5 Anfragen pro Sekunde je IP-Adresse, kurzzeitig bis 60 auf einmal, dazu das Minutenlimit von 6000 je Token, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Gewöhnliches Verschieben und Zoomen bleibt deutlich darunter; erst wer dauerhaft schneller neue Kacheln anfordert, bekommt `429`.

## Beispiele

Die Beispiele mit Leaflet-Fassung stehen in der [Übersicht](https://docs.mapx.at/de/examples/index.md) (Filter „Leaflet“), etwa [Karte einbetten](https://docs.mapx.at/de/examples/embed-map.md#leaflet) und [Route](https://docs.mapx.at/de/examples/route.md#leaflet).

# Static Tiles

Gerenderte Rasterkacheln für Karte, Luftbild und Gelände, für Leaflet, OpenLayers und QGIS.

Der Endpunkt liefert fertig gerenderte Rasterkacheln eines Stils, in der Form der Mapbox Static Tiles API. Damit lassen sich Karten, Luftbild und Gelände in Clients ohne Vektor-Unterstützung zeigen (Leaflet, OpenLayers, QGIS, Karten in Berichten). Ohne Schlüssel, CORS offen.

**GET https://api.mapx.at/v1/styles/{id}/tiles/{tilesize}/{z}/{x}/{y}[@2x][.png|.webp]**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Stil: <code>karte</code>, <code>karte-morgen</code>, <code>karte-abend</code>, <code>karte-nacht</code>, <code>luftbild</code>, <code>hybrid</code> oder <code>gelaende</code>." },
  { name: "tilesize", type: "integer", required: true, desc: "<code>256</code> (Standard-XYZ, Kartenzoom <code>z-1</code>) oder <code>512</code> (Kartenzoom <code>z</code>, Beschriftung wirkt kleiner)." },
  { name: "z, x, y", type: "integer", required: true, desc: "Kachel im XYZ-Schema (Ursprung oben links)." },
  { name: "@2x", type: "suffix", desc: "Doppelte Pixelzahl (512 bzw. 1024 Pixel) bei gleichem Ausschnitt." },
  { name: "png / webp", type: "suffix", desc: "Bildformat, Standard PNG." },
]} />

## Kachelränder

Der Ausschnitt wird mit einem Rand von 96 Pixeln gerendert und auf die Kachel zugeschnitten. Dadurch laufen Linien, Flächen und Schummerung an den Rändern ohne Naht weiter und Beschriftungen werden nicht an der Kachelkante abgeschnitten. Die Platzierung der Beschriftungen wird je Kachel einzeln berechnet; ein Name nahe der Kante kann daher in der Nachbarkachel an etwas anderer Stelle stehen. Für Beschriftungen ohne Doppelungen den Stil als Vektorkarte (<a href="/api/styles/">Styles</a>) verwenden.

## Antworten und Limits

`404` für Kacheln ohne Daten (außerhalb Österreichs) und unter dem Mindestzoom (`luftbild`, `hybrid` und `gelaende` ab Kartenzoom 5), ungültige Größe, Endung oder Kachelnummer sowie unbekannte Stile. `ETag` und `Cache-Control: public, max-age=86400`, `If-None-Match` ergibt `304`. Das Rendern ist aufwendig und zählt im Rate-Limit mehrfach (<a href="/api/fehler/">Fehler und Limits</a>); wiederholte Abrufe bedient der Cache.

## Leaflet

```js
L.tileLayer('https://api.mapx.at/v1/styles/karte/tiles/256/{z}/{x}/{y}@2x', {
  tileSize: 256,
  detectRetina: false, // @2x ist schon in der URL / @2x is already in the URL
  maxZoom: 22,
  attribution: '© mapx.at, BEV, GIP.at'
}).addTo(map);
```

Für 512er-Kacheln `tileSize: 512` und `zoomOffset: -1` setzen und `/tiles/512/` verwenden.

## QGIS

Browser, Rechtsklick auf **XYZ Tiles**, **Neue Verbindung**:

| | |
|---|---|
| Name | mapx Karte |
| URL | `https://api.mapx.at/v1/styles/karte/tiles/256/{z}/{x}/{y}` |
| Max. Zoom | 22 |

Für `luftbild` und `gelaende` Min. Zoom 5 eintragen. Die Ebene lässt sich wie jede Rasterebene in Layouts und Berichten verwenden.

## Einzelne Kachel

```sh
curl -o kachel.png 'https://api.mapx.at/v1/styles/luftbild/tiles/256/15/17767/11367'
```
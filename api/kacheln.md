# Kacheln

Einzelne Kacheln über /v1/tiles/{tileset}/{z}/{x}/{y}.{ext} oder die ganze PMTiles-Datei per HTTP-Range.

Alle Produkte gibt es auf zwei Wegen: als einzelne Kachel (XYZ, für jeden Kartenclient) und als PMTiles-Datei (eine Datei, der Client liest per HTTP-Range nur die nötigen Kacheln). Inhalt und Qualität sind gleich.

| | XYZ | PMTiles |
|---|---|---|
| Geeignet für | Leaflet, OpenLayers, QGIS, Apps, Server | MapLibre, eigenes Hosting, offline |
| Adresse | `/v1/tiles/{tileset}/{z}/{x}/{y}.{ext}` | `<datei>.pmtiles` |
| Stabilität | Tileset-Name zeigt immer auf den aktuellen Stand | Datei mit Stichtag im Namen, ändert sich nie |

## Kachel abrufen

**GET https://tiles.mapx.at/v1/tiles/{tileset}/{z}/{x}/{y}.{ext}**

<ParamTable params={[
  { name: "tileset", type: "string", required: true, desc: "Produkt.", values: [
    { v: "dtm", d: "<a href=\"/daten/dtm/\">DTM</a>, Terrain-RGB, <code>.webp</code> (auch <code>terrain-rgb</code>, früher <code>relief</code>)" },
    { v: "dsm", d: "<a href=\"/daten/dsm/\">DSM</a>, Terrain-RGB, <code>.webp</code> (auch <code>relief-dsm-terrainrgb</code>)" },
    { v: "orthophoto", d: "<a href=\"/daten/orthophoto/\">Orthophoto</a>, Bild, <code>.webp</code>" },
    { v: "streets", d: "<a href=\"/daten/basiskarte/\">Basiskarte</a>, Vektor, <code>.pbf</code>" },
    { v: "contours", d: "<a href=\"/daten/hoehenlinien/\">Höhenlinien</a>, Vektor, <code>.pbf</code>" },
    { v: "cadastre", d: "<a href=\"/daten/kataster/\">Kataster</a>, Vektor, <code>.pbf</code>" },
    { v: "orthophoto-flight-blocks", d: "<a href=\"/daten/flugbloecke/\">Flugblöcke</a>, Vektor, <code>.pbf</code>" }
  ] },
  { name: "z", type: "integer", required: true, desc: "Zoomstufe im Bereich des Produkts (siehe Kennwerte der Produktseite). Außerhalb antwortet der Dienst mit <code>404</code>." },
  { name: "x", type: "integer", required: true, desc: "Spalte, 0 bis 2<sup>z</sup>−1 von West nach Ost." },
  { name: "y", type: "integer", required: true, desc: "Zeile, 0 bis 2<sup>z</sup>−1 von Nord nach Süd (XYZ-Schema, nicht TMS)." },
  { name: "ext", type: "string", required: true, desc: "Dateiendung, muss zum Produkt passen, sonst <code>404</code>.", values: [
    { v: "webp", d: "Bild- und Höhenkacheln (DTM, DSM verlustfrei; Orthophoto)" },
    { v: "pbf", d: "Vektorkacheln (Mapbox Vector Tile 2.1), gzip, wenn der Client es annimmt" },
  ] },
]} />

### Beispiel

<Tabs syncKey="client">
  <TabItem label="curl">
<Code lang="bash" code={`curl -s -o tile.webp https://tiles.mapx.at/v1/dtm/12/2192/1439.webp
curl -s --compressed -o tile.pbf https://tiles.mapx.at/v1/streets/14/8860/5730.pbf`} />
  </TabItem>
  <TabItem label="Leaflet">
<Code lang="js" code={`L.tileLayer("https://tiles.mapx.at/v1/orthophoto/{z}/{x}/{y}.webp", {
  tileSize: 512, zoomOffset: -1, maxNativeZoom: 18, maxZoom: 20,
  attribution: "Orthophoto: BEV, CC BY 4.0"
}).addTo(map);`} />
  </TabItem>
  <TabItem label="OpenLayers">
<Code lang="js" code={`new ol.layer.Tile({ source: new ol.source.XYZ({
  url: "https://tiles.mapx.at/v1/orthophoto/{z}/{x}/{y}.webp", tileSize: 512, maxZoom: 18
}) });`} />
  </TabItem>
</Tabs>

### Antwort

| Header | Bedeutung |
|---|---|
| `Content-Type` | `image/webp` oder `application/vnd.mapbox-vector-tile` |
| `ETag` | Version der Kachel; mit <code>If-None-Match</code> antwortet der Dienst <code>304</code> |
| `Cache-Control` | `public, max-age=3600` |

Gibt es an dieser Stelle keine Daten, antwortet der Dienst mit <code>404</code> als <code>problem+json</code> (wie Mapbox, nicht <code>204</code>). Clients behandeln das als leere Kachel.

## PMTiles-Datei

**GET https://tiles.mapx.at/<datei>.pmtiles**

Die Datei unterstützt HTTP-Range-Requests. MapLibre liest sie mit dem [pmtiles](https://docs.protomaps.com/pmtiles/maplibre)-Protokoll direkt aus dem Browser; der aktuelle Dateiname steht auf jeder Produktseite. Weil sich eine Datei nie ändert, kann sie unbegrenzt gecacht oder gespiegelt werden.

<Code lang="js" code={`import { Protocol } from "pmtiles";
maplibregl.addProtocol("pmtiles", new Protocol().tile);
// source: { type: "raster-dem", url: "pmtiles://https://tiles.mapx.at/<file>.pmtiles", encoding: "mapbox", tileSize: 512 }`} />

Eigenes Hosting: [PMTiles selbst hosten](/anleitungen/pmtiles-selbst-hosten/).
# TileJSON

TileJSON 3.0 je Produkt unter /v1/tiles/{tileset}.json: Kachel-URL, Zoombereich, Gebiet und Attribution in einer Antwort.

TileJSON beschreibt ein Produkt so, dass Kartenclients es ohne weitere Angaben einbinden: MapLibre, Mapbox GL, OpenLayers und QGIS lesen Zoombereich, Gebiet und Attribution daraus. Für Produkte mit neuem Stand bleibt die URL gleich.

## TileJSON abrufen

**GET https://tiles.mapx.at/v1/tiles/{tileset}.json**

<ParamTable params={[
  { name: "tileset", type: "string", required: true, desc: "Produkt, Werte wie bei <a href=\"/api/kacheln/\">Kacheln</a>." },
]} />

<Code lang="js" code={`map.addSource("ortho", { type: "raster", url: "https://tiles.mapx.at/v1/orthophoto.json", tileSize: 512 });`} />

## Antwort

<ParamTable kind="response" params={[
  { name: "tilejson", type: "string", desc: "Version der Spezifikation: <code>3.0.0</code>" },
  { name: "tiles", type: "string[]", desc: "Kachel-URL mit Platzhaltern <code>{z}/{x}/{y}</code>" },
  { name: "minzoom, maxzoom", type: "integer", desc: "Zoombereich mit gespeicherten Kacheln; darüber zoomt der Client auf" },
  { name: "bounds", type: "number[4]", desc: "Gebiet <code>[west, süd, ost, nord]</code> in Grad" },
  { name: "center", type: "number[3]", desc: "Vorschlag für Startansicht <code>[lon, lat, zoom]</code>" },
  { name: "scheme", type: "string", desc: "Kachelschema", values: [{ v: "xyz", d: "Zeile 0 im Norden" }] },
  { name: "tileSize", type: "integer", desc: "Kachelgröße in Pixeln bei Raster- und Höhenkacheln: <code>512</code>" },
  { name: "encoding", type: "string", desc: "Nur Höhenkacheln: Kodierung der Höhe", values: [{ v: "mapbox", d: "<a href=\"/konzepte/terrain-rgb/\">Terrain-RGB</a>, 0,1 m" }] },
  { name: "vector_layers", type: "object[]", desc: "Nur Vektorkacheln: Layer mit <code>id</code>, <code>minzoom</code>, <code>maxzoom</code> und <code>fields</code> (Attributname → Typ)" },
  { name: "attribution", type: "string", desc: "Pflichttext nach CC BY 4.0 als HTML" },
  { name: "name, description, version", type: "string", desc: "Name, Beschreibung und Stand des Produkts" },
]} />

<Code lang="json" code={`{
  "tilejson": "3.0.0",
  "name": "relief-terrainrgb",
  "tiles": ["https://tiles.mapx.at/v1/tiles/terrain-rgb/{z}/{x}/{y}.webp"],
  "minzoom": 5, "maxzoom": 16,
  "bounds": [9.5306, 46.3697, 17.1661, 49.0235],
  "scheme": "xyz", "tileSize": 512, "encoding": "mapbox",
  "attribution": "Relief: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0"
}`} />

Die Antwort trägt <code>ETag</code> und <code>Cache-Control</code>.
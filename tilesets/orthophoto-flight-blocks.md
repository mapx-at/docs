# Flight blocks

Which flight supplies which orthophoto pixel: winning areas of the BEV surveys with flight period, as vector tiles z6 to z17.

export const t = data.tilesets.dop_flugblock;
export const url = `https://tiles.mapx.at/${t.datei}`;

For each pixel of the [orthophoto](/tilesets/orthophoto/), the area of the survey flight (operat) it comes from, with flight period.

<Nutzungen items={[{t:"Show capture date",d:"\"Imagery from\" in the map"},{t:"Judge currency",d:"Age of the aerial image at a location"},{t:"Explain image changes",d:"Understand color seams between flights"},{t:"Quality checks",d:"Which flight covers which area"}]} />

## Specifications

<Kennwerte id="dop_flugblock" lang="en" />

## Live map

<ProduktKarte id="dop_flugblock" center={[14.3, 47.6]} zoom={7} lang="en" />

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json" } },
    layers: [{ id: "dop_flugblock", type: "line", source: "v", "source-layer": "dop_flugblock", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/{z}/{x}/{y}.pbf`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/tiles/orthophoto-flight-blocks/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Layer reference

### dop_flugblock

<FeldReferenz ts="dop_flugblock" layer="dop_flugblock" />

## Source and currency

- Source: BEV "Currency of orthophoto surveys, color and infrared" (Aktualität Orthophoto Operate Farbe und Infrarot).
- The areas show where a survey is actually visible in the orthophoto. A survey can have several areas.

## Notes

- Date information is the period of the survey, not a daily date.
- A flight block is an operat, not a single flight day: there is no daily date, only the period of the operat; an operat can last up to three months.


## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.dop_flugblock.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/reference/attribution/).
# Cadastre

Parcel boundaries from the BEV Digital Cadastral Map as vector tiles, z16, with cadastral municipality and parcel number.

export const t = data.tilesets.kataster;
export const url = `https://tiles.mapx.at/${t.datei}`;

The boundaries of all parcels as lines, with cadastral municipality number and parcel number, from the BEV Digital Cadastral Map. The tileset has z16 only; clients overzoom above it.

<Nutzungen items={[{t:"Parcel search",d:"Boundaries and numbers on the map"},{t:"Planning and preliminary checks",d:"Parcel boundaries over the orthophoto"},{t:"Real estate and site maps",d:"Parcels as a map layer"},{t:"Overlays",d:"Cadastre over the basemap or aerial imagery",href:"/guides/style-the-basemap/"},{t:"QGIS",d:"Vector tiles in desktop GIS",href:"/guides/qgis/"}]} />

## Specifications

<Kennwerte id="kataster" lang="en" />

## Live map

<ProduktKarte id="kataster" center={[15.334, 48.227]} zoom={16.5} lang="en" />

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" } },
    layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution. Send the token with every request: add the `transformRequest` from [First map](/start/first-map/) to the map options. With PMTiles directly use `url: "pmtiles://${url}?access_token=YOUR_TOKEN"` (protocol as in the [PMTiles guide](/guides/self-host-pmtiles/)).
  </TabItem>
  <TabItem label="QGIS">
    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/cadastre/{z}/{x}/{y}.pbf?access_token=YOUR_TOKEN`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s -H "Authorization: Bearer YOUR_TOKEN" https://tiles.mapx.at/v1/tiles/cadastre.json
curl -s -H "Authorization: Bearer YOUR_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/cadastre/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

Because only z16 exists, set `minzoom: 16` in the style; the source with `maxzoom: 16` overzooms automatically.

## Layer reference

The lines are boundaries, not areas: a click hits the boundary line and returns the parcel it belongs to. Owner and area are not included.

### parcel_l

<FeldReferenz ts="kataster" layer="parcel_l" />

## Source and currency

- Source: BEV **Digital Cadastral Map (DKM)**, reference date 1 Apr 2026.

## Notes

> The display is not legally binding. For boundary questions the cadastre at the survey office applies.

- The display is not legally binding; the depiction follows the digital cadastral map.
- There are no tiles below z16.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.kataster.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/reference/attribution/).
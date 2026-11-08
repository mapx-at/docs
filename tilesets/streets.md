# Base map

Vector base map of Austria from open government data: land use, buildings, roads, rail, water, boundaries, names, house numbers. z0 to z16.

export const t = data.tilesets.basis;
export const url = `https://tiles.mapx.at/${t.datei}`;

The basemap combines open data from BEV, GIP, Umweltbundesamt and BMLUK in 18 layers. It contains no OpenStreetMap data. Cadastre, contour lines and relief are separate products and can be layered on top.

<Nutzungen items={[{t:"Custom map style",d:"Style the layers as you like",href:"/guides/style-the-basemap/"},{t:"Web maps and apps",d:"Vector map with your own design up to z16"},{t:"Planning and navigation background",d:"Roads, rail, water, boundaries"},{t:"House numbers and names",d:"Addresses, places, peaks and water names"},{t:"Overlays",d:"Cadastre, contours and relief on top",href:"/guides/overlay-contours/"}]} />

## Specifications

<Kennwerte id="basis" lang="en" />

## Live map

<ProduktKarte id="basis" center={[15.334, 48.227]} zoom={14} lang="en" />

The live map shows the layers with automatic colors; your own style defines the final look. Clicking shows the attributes.

## Add to your map

<Tabs syncKey="client">
  <TabItem label="MapLibre">
    <Code lang="js" code={`new maplibregl.Map({
  container: "map",
  style: {
    version: 8,
    sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
    layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
  }
});`} />
    TileJSON provides zoom range, bounds and attribution; with PMTiles directly use `url: "pmtiles://${url}"` (protocol as in the [Quickstart](/start/)).
  </TabItem>
  <TabItem label="QGIS">
    Vector Tiles → New Connection, URL `https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf`, leave the style URL empty. Alternatively open the PMTiles file directly as a layer (QGIS 3.32 and later reads PMTiles over HTTP).
  </TabItem>
  <TabItem label="curl">
    <Code lang="bash" code={`curl -s https://tiles.mapx.at/v1/tiles/streets.json
curl -s -o kachel.pbf https://tiles.mapx.at/v1/tiles/streets/14/8860/5730.pbf`} />
  </TabItem>
</Tabs>

## Layer reference

A layer name ending in `_a` is an area, `_l` a line, `_p` a point. `min_zoom` on a feature says from which level it is intended; `sort_rank` orders within the layer.

<LayerTabelle id="basis" lang="en" info={{'landuse_a': 'Land use from the DKM: residential, garden, farmland, vineyard, alpine_meadow, forest, glacier, rock, water, wetland, transport, quarry, cemetery, recreation', 'building_a': 'Buildings (DKM) with height from the DLM; <code>kind</code> building or church, school, hospital, station, castle, hut …; merged at z12–14', 'road_l': 'Roads and paths (GIP): highway, major_road, minor_road, service, track, path; <code>structure</code> bridge/tunnel', 'rail_l': 'Rail (GIP): rail, narrow_gauge, rack, subway, tram, funicular, ferry', 'aerialway_l': 'Aerial ropeways and lifts (DLM): cable_car, gondola, chair_lift, drag_lift …', 'transit_p': 'Railway stations, stops, lift stations', 'airport_p': 'Airfields (Wikidata): airport, airfield, heliport, military', 'ski_a': 'Ski areas (Umweltbundesamt, as of 2017)', 'boundary_l': 'Boundaries (BEV VGD): country, state, district, municipality; <code>left</code>/<code>right</code> area on the left/right', 'boundary_label_p': 'Names of state, district, municipality; <code>code</code> identifier', 'place_p': 'Places (BEV Geographical Names): city, town, village, hamlet, suburb, locality; <code>capital</code>', 'natural_p': 'Mountains, saddles, valleys, field names, castles, ruins …', 'poi_p': 'Churches, summit crosses and facilities (BEV DLM, in Vienna additionally Wien-OGD): schools, hospitals, huts, petrol stations, museums …', 'housenumber_p': 'House numbers (BEV address register): <code>kind</code> building = building coordinate, parcel = parcel address', 'water_l': 'Flowing waters (GGN): river, stream, canal, ditch', 'water_a': 'Standing waters (GGN): lake, reservoir, pond', 'water_label_l': 'Water names along the line', 'water_label_p': 'Lake names, one point per lake'}} />

Every field and value of each layer: [Basemap: layer reference](/reference/style-spec/).

## Source and currency

| Topic | Source | As of |
|---|---|---|
| Land use, buildings | BEV Digital Cadastral Map (DKM), heights from the DLM | 1 Apr 2026 |
| Roads, rail | GIP-OGD (Graph Integration Platform) | 30 Apr 2026 |
| Lifts, stations, POI | BEV DLM, Wien-OGD (Vienna only) | |
| Boundaries | BEV administrative boundaries (VGD) | 1 Oct 2026 |
| Places, names, summits | BEV Geographical Names | 23 Mar 2026 |
| House numbers | BEV address register | 1 Oct 2026 |
| Water | BMLUK Gesamtgewässernetz (GGN v19) | |
| Ski areas | Umweltbundesamt | 2017 |
| Airfields | Wikidata (CC0) | |


## Notes

- Not included: stops of private railways, subway and tram stops; ski areas as of 2017; small airfields only in part.
- Abroad the map is practically empty (only Lake Constance and waters up to just beyond the border).
- mapx does not provide glyphs and fonts for labels; the style needs its own.

## Attribution

Required under CC BY 4.0. MapLibre picks up the text automatically from TileJSON or PMTiles:

<Code lang="html" code={data.tilesets.basis.metadata.attribution || "BEV, CC BY 4.0"} />

More: [License](/reference/attribution/).
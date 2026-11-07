# Styles

Ready-made MapLibre styles for map, aerial imagery and terrain at /v1/styles/{id}.

A style is a complete [MapLibre style document](https://maplibre.org/maplibre-style-spec/): sources (PMTiles from `tiles.mapx.at`), layers, fonts and symbols. The styles need the `pmtiles://` protocol in the client ([Quick start](/en/start/)).

**GET https://api.mapx.at/v1/styles**

Lists all styles as JSON (`id` and `url`).

**GET https://api.mapx.at/v1/styles/{id}**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Style", values: [
    { v: "karte", d: "Vector map (roads, buildings, water, forest, names), daylight" },
    { v: "karte-morgen", d: "Map in morning light" },
    { v: "karte-abend", d: "Map in evening light" },
    { v: "karte-nacht", d: "Dark map for the night" },
    { v: "luftbild", d: "Orthophoto with labels" },
    { v: "gelaende", d: "Map with hillshade and contour lines" },
  ] },
]} />

The response is `application/json` and can be passed directly as `style` to `new maplibregl.Map()`.

<Code lang="js" code={`const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/karte",
  center: [13.04, 47.80], zoom: 12,
});`} />

Live: [Embed a map](/en/beispiele/karte-einbetten/), [Switch the basemap](/en/beispiele/basiskarte-wechseln/), [Time of day and night mode](/en/beispiele/licht/).
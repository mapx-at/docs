# Styles

Ready-made MapLibre styles for map, aerial imagery and terrain at /v1/styles/{id}.

A style is a complete [MapLibre style document](https://maplibre.org/maplibre-style-spec/): sources (PMTiles from `tiles.mapx.at`), layers, fonts and symbols. The styles need the `pmtiles://` protocol in the client ([Quick start](/en/start/)).

**GET https://api.mapx.at/v1/styles**

Lists all styles as JSON (`id` and `url`).

**GET https://api.mapx.at/v1/styles/{id}**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Style", values: [
    { v: "streets", d: "Vector map (roads, buildings, water, forest, names), daylight" },
    { v: "streets-dawn", d: "Map in morning light" },
    { v: "streets-dusk", d: "Map in evening light" },
    { v: "streets-night", d: "Dark map for the night" },
    { v: "traffic-day", d: "Daytime map with the traffic layer visible" },
    { v: "satellite", d: "Orthophoto with labels" },
    { v: "satellite-streets", d: "Orthophoto with roads, names and boundaries" },
    { v: "outdoors", d: "Map with hillshade and contour lines" },
  ] },
]} />

> The German style IDs used until November 2026 (`karte`, `karte-morgen`, `karte-abend`, `karte-nacht`, `karte-verkehr`, `luftbild`, `hybrid`, `gelaende`) keep working as aliases until **2027-05-31**. Their responses carry the headers `Deprecation`, `Sunset` and `Link: …; rel="successor-version"`. All names: [Migration to English names](/en/api/migration/).

The response is `application/json` and can be passed directly as `style` to `new maplibregl.Map()`.

<Code lang="js" code={`const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets",
  center: [13.04, 47.80], zoom: 12,
});`} />

Live: [Embed a map](/en/beispiele/karte-einbetten/), [Switch the basemap](/en/beispiele/basiskarte-wechseln/), [Time of day and night mode](/en/beispiele/licht/).
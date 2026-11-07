# Styles

Fertige MapLibre-Stile für Karte, Luftbild und Gelände unter /v1/styles/{basis}.

> `/v1/styles/{basis}` wird gerade ausgerollt. Bis dahin liegen dieselben Stile als Dateien unter `https://mapx.at/styles/`: `standard-tag-normal.json` (Karte), `hybrid.json` (Luftbild), `gelaende.json` (Gelände), dazu `standard-morgen|abend|nacht-normal.json` für andere Lichtstimmungen.

Ein Stil ist ein vollständiges [MapLibre-Style-Dokument](https://maplibre.org/maplibre-style-spec/): Quellen (PMTiles von `tiles.mapx.at`), Ebenen, Schriften und Symbole. Die Stile benötigen das `pmtiles://`-Protokoll im Client ([Schnellstart](/start/)).

**GET https://api.mapx.at/v1/styles/{basis}**

<ParamTable params={[
  { name: "basis", type: "string", required: true, desc: "Basiskarte", values: [
    { v: "karte", d: "Vektorkarte (Straßen, Gebäude, Gewässer, Wald, Namen)" },
    { v: "luftbild", d: "Orthophoto mit Beschriftung" },
    { v: "gelaende", d: "Karte mit Schummerung und Höhenlinien" },
  ] },
]} />

Die Antwort ist `application/json` und lässt sich direkt als `style` an `new maplibregl.Map()` übergeben.

<Code lang="js" code={`const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/karte",
  center: [13.04, 47.80], zoom: 12,
});`} />

Live: [Karte einbetten](/beispiele/karte-einbetten/), [Basiskarte wechseln](/beispiele/basiskarte-wechseln/), [Tageszeit und Nachtmodus](/beispiele/licht/).
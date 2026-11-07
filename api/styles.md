# Styles

Fertige MapLibre-Stile für Karte, Luftbild und Gelände unter /v1/styles/{id}.

Ein Stil ist ein vollständiges [MapLibre-Style-Dokument](https://maplibre.org/maplibre-style-spec/): Quellen (PMTiles von `tiles.mapx.at`), Ebenen, Schriften und Symbole. Die Stile benötigen das `pmtiles://`-Protokoll im Client ([Schnellstart](/start/)).

**GET https://api.mapx.at/v1/styles**

Listet alle Stile als JSON (`id` und `url`).

**GET https://api.mapx.at/v1/styles/{id}**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Stil", values: [
    { v: "karte", d: "Vektorkarte (Straßen, Gebäude, Gewässer, Wald, Namen), Tageslicht" },
    { v: "karte-morgen", d: "Karte in Morgenstimmung" },
    { v: "karte-abend", d: "Karte in Abendstimmung" },
    { v: "karte-nacht", d: "Dunkle Karte für die Nacht" },
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
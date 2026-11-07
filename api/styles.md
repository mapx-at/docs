# Styles

Fertige MapLibre-Stile für Karte, Luftbild und Gelände unter /v1/styles/{id}.

Ein Stil ist ein vollständiges [MapLibre-Style-Dokument](https://maplibre.org/maplibre-style-spec/): Quellen (PMTiles von `tiles.mapx.at`), Ebenen, Schriften und Symbole. Die Stile benötigen das `pmtiles://`-Protokoll im Client ([Schnellstart](/start/)).

**GET https://api.mapx.at/v1/styles**

Listet alle Stile als JSON (`id` und `url`).

**GET https://api.mapx.at/v1/styles/{id}**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Stil", values: [
    { v: "streets", d: "Vektorkarte (Straßen, Gebäude, Gewässer, Wald, Namen), Tageslicht" },
    { v: "streets-dawn", d: "Karte in Morgenstimmung" },
    { v: "streets-dusk", d: "Karte in Abendstimmung" },
    { v: "streets-night", d: "Dunkle Karte für die Nacht" },
    { v: "traffic-day", d: "Karte bei Tag mit sichtbarer Ebene Verkehr" },
    { v: "satellite", d: "Orthophoto mit Beschriftung" },
    { v: "satellite-streets", d: "Orthophoto mit Straßen, Namen und Grenzen" },
    { v: "outdoors", d: "Karte mit Schummerung und Höhenlinien" },
  ] },
]} />

> Die bis November 2026 verwendeten deutschen Stil-IDs (`karte`, `karte-morgen`, `karte-abend`, `karte-nacht`, `karte-verkehr`, `luftbild`, `hybrid`, `gelaende`) funktionieren noch bis **31.05.2027** als Alias. Die Antwort trägt dann die Header `Deprecation`, `Sunset` und `Link: …; rel="successor-version"`. Alle Namen: [Migration auf englische Namen](/api/migration/).

Die Antwort ist `application/json` und lässt sich direkt als `style` an `new maplibregl.Map()` übergeben.

<Code lang="js" code={`const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets",
  center: [13.04, 47.80], zoom: 12,
});`} />

Live: [Karte einbetten](/beispiele/karte-einbetten/), [Basiskarte wechseln](/beispiele/basiskarte-wechseln/), [Tageszeit und Nachtmodus](/beispiele/licht/).
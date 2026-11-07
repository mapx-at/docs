# Schriften (Fonts)

Glyphen als Protobuf für MapLibre und Mapbox GL, mit Kombination mehrerer Schriften zu einem Fontstack.

Die Schriften für Beschriftungen, in der Form der Mapbox Fonts API. Ohne Schlüssel, CORS offen.

**GET https://api.mapx.at/v1/fonts**

Liste der verfügbaren Schriften, z. B. `Noto Sans Regular`, `Noto Sans Bold`, `Open Sans Regular`.

**GET https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf**

<ParamTable params={[
  { name: "fontstack", type: "string", required: true, desc: "Eine Schrift oder mehrere, kommagetrennt (Leerzeichen als <code>%20</code>), höchstens 8. Je Zeichen gewinnt die erste Schrift, die es enthält (Fallback wie in Mapbox)." },
  { name: "range", type: "string", required: true, desc: "<code>0-255</code>, <code>256-511</code>, …: 256 Zeichen ab einem Vielfachen von 256, bis <code>65280-65535</code>." },
]} />

Antwort ist `application/x-protobuf` mit `ETag` und `Cache-Control: public, max-age=86400`. Unbekannte Schrift: `404`, ungültiger Bereich oder Name: `400`.

## Im Stil

MapLibre und Mapbox GL fragen die Bereiche selbst ab, wenn der Stil ein `glyphs`-Feld hat. Die Stile aus <a href="/api/styles/">/v1/styles</a> enthalten es bereits:

```json
{
  "glyphs": "https://api.mapx.at/v1/fonts/{fontstack}/{range}.pbf",
  "layers": [{ "type": "symbol", "layout": { "text-font": ["Noto Sans Regular", "Open Sans Regular"], "text-field": "{name}" } }]
}
```

`text-font` mit zwei Einträgen ergibt den Fontstack `Noto Sans Regular,Open Sans Regular`.
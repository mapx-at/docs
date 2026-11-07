# Sprites

Icon-Atlanten (Sprite-Index als JSON und Bild als PNG) für die Stile, normal und @2x, dazu SDF-Icons.

Die Icons der Karte, in der Form der Mapbox Sprites API. Ohne Schlüssel, CORS offen.

**GET https://api.mapx.at/v1/styles/{id}/sprite[@2x].{json,png}**

**GET https://api.mapx.at/v1/styles/{id}/sprite-sdf[@2x].{json,png}**

**GET https://api.mapx.at/v1/sprites**

<ParamTable params={[
  { name: "id", type: "string", required: true, desc: "Stil aus <a href=\"/api/styles/\">/v1/styles</a> (<code>streets</code>, <code>satellite</code>, …). Unbekannter Stil: <code>404</code>." },
  { name: "@2x", type: "suffix", desc: "Doppelte Auflösung für Bildschirme mit hoher Pixeldichte." },
  { name: "json / png", type: "suffix", required: true, desc: "<code>.json</code> ist der Index (Name, Position, Größe, <code>pixelRatio</code>), <code>.png</code> der Atlas." },
]} />

`sprite` enthält die farbigen Icons, `sprite-sdf` einfarbige SDF-Icons, die der Stil per `icon-color` einfärbt. Aktuell verwenden alle Stile dieselben Sprites; `/v1/sprites` listet sie, `/v1/sprites/{datei}` liefert sie ohne Stilbezug. Antworten tragen `ETag` und `Cache-Control: public, max-age=86400`.

## Im Stil

Die Stile aus <a href="/api/styles/">/v1/styles</a> verweisen auf diese Adressen; MapLibre hängt `.json`, `.png` und `@2x` selbst an:

```json
{
  "sprite": [
    { "id": "default", "url": "https://api.mapx.at/v1/styles/streets/sprite" },
    { "id": "sdf", "url": "https://api.mapx.at/v1/styles/streets/sprite-sdf" }
  ]
}
```

Icons des zweiten Sprites heißen im Stil `sdf:<name>`.
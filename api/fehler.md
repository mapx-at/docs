# Fehler und Limits

Fehlerformat nach RFC 9457, Statuscodes, Anfragelimits und Caching aller mapx-APIs.

## Fehlerformat

Alle Endpunkte antworten bei Fehlern mit `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)), nie mit HTML.

<ParamTable kind="response" params={[
  { name: "type", type: "string", desc: "Fehlertyp als URI: <code>https://docs.mapx.at/api/errors/#&lt;code&gt;</code>" },
  { name: "title", type: "string", desc: "Kurze Zusammenfassung, englisch (deutsch mit <code>Accept-Language: de</code>)" },
  { name: "status", type: "integer", desc: "HTTP-Status" },
  { name: "detail", type: "string", desc: "Erklärung, was an der Anfrage falsch ist" },
  { name: "code", type: "string", desc: "Stabiler Fehlercode, siehe <a href=\"/api/errors/\">Fehlerkatalog</a>" },
  { name: "param", type: "string", desc: "Betroffener Parameter (wo sinnvoll)" },
  { name: "allowed", type: "array", desc: "Erlaubte Werte bei Aufzählungen (wo sinnvoll)" },
]} />

<Code lang="json" code={`{ "type": "https://docs.mapx.at/api/errors/#unknown-path", "title": "Unknown path", "status": 404, "detail": "Unknown path", "code": "unknown-path" }`} />

Alle Codes mit Bedeutung und Abhilfe: [Fehlerkatalog](/api/errors/).

## Statuscodes

| Status | Bedeutung | Was tun |
|---|---|---|
| `200` | Erfolg | |
| `304` | Nicht geändert (bei <code>If-None-Match</code>) | Kopie aus dem Cache verwenden |
| `400` | Parameter fehlen oder sind ungültig | <code>detail</code> lesen, Anfrage korrigieren |
| `404` | Produkt unbekannt, falsche Endung, keine Kachel an dieser Stelle oder Zoomstufe | bei Kacheln als leere Kachel behandeln |
| `422` | Anfrage gültig, aber nicht beantwortbar (etwa keine Route) | Punkte oder Zeit ändern |
| `429` | Limit überschritten | <code>Retry-After</code> Sekunden warten, dann mit wachsendem Abstand erneut |
| `503` | Vorübergehend nicht verfügbar | später erneut versuchen |

## Limits

Limits gelten je IP-Adresse und Zone. Jede Antwort trägt `RateLimit-Policy`, zum Beispiel `"search";q=10;w=1;burst=30` (10 Anfragen je Sekunde, dazu 30 Spielraum). Zonen: Kacheln, Schriften, Sprites 100/s (Spielraum 200); Suche, Geocoding, Search Box, Autofill, Abfahrten, Wetter, Verkehr 10/s (30); Routing 5/s (20); Höhe 10/s (20); Static Tiles und Verkehrskacheln 5/s (60); aufwendige Endpunkte (Static Images, Isochrone, Matrix, Matching, Optimized Trips, Optimization, Batch) 30 pro Minute (3 bis 5).

| API | Anfragen | Je Anfrage |
|---|---|---|
| Kacheln, TileJSON | 50 pro Sekunde, kurzzeitig 300 | eine Kachel |
| PMTiles | wie Kacheln | beliebige Range-Requests |
| Höhe und Profil | 10 pro Sekunde, kurzzeitig 20; je 32 Punkte zählt eine Anfrage mehr | bis 512 Punkte, bis 1024 Profilpunkte |

Für mehr Last (Batch-Verarbeitung, ganze Gebiete) ist die PMTiles-Datei der richtige Weg: einmal herunterladen oder spiegeln, dann lokal lesen.

## Caching

| Header | Wert |
|---|---|
| `Cache-Control` | `public, max-age=3600` (Kacheln, TileJSON, Höhe, Grundstück), `max-age=300` (Geocoding, Reverse, Autofill-Vorschläge, Address Validation per GET, Tileset-Liste), `max-age=0` (Routing, Matrix, Isochrone, POST-Anfragen) |
| `ETag`, `Last-Modified` | Version; mit `If-None-Match` bzw. `If-Modified-Since` → `304` (Kacheln, Stile, TileJSON, OpenAPI) |
| `Server-Timing` | `router`, `search`, `render`, `total` in Millisekunden |
| `RateLimit-Policy` | Zone und Limit des Endpunkts |

PMTiles-Dateien tragen den Stichtag im Namen und ändern sich nie; sie dürfen unbegrenzt gecacht werden.

## Attribution

JSON-Antworten mit Daten tragen ein Feld `attribution` auf oberster Ebene (Text, ohne HTML) mit der Quelle, die bei Anzeige oder Weitergabe zu nennen ist, etwa `Datenquelle: BEV – DKM, CC BY 4.0` beim Grundstück oder `Datenquelle: GIP.at …` bei Routing, Matrix und Isochrone. Ausnahmen: die Listen `/v1/styles`, `/v1/fonts`, `/v1/sprites` (Arrays); Stile nennen die Quellen in ihren `sources`. Siehe [Attribution](/anleitungen/attribution/).

## Betrieb

- `GET https://api.mapx.at/health` – Lebenszeichen für Monitoring.
- `GET https://api.mapx.at/v1/openapi.json` – OpenAPI-3.1-Beschreibung (englisch, mit Beispielen).
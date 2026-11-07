# Fehler und Limits

Fehlerformat nach RFC 9457, Statuscodes, Anfragelimits und Caching aller mapx-APIs.

## Fehlerformat

Alle Endpunkte antworten bei Fehlern mit `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)), nie mit HTML.

<ParamTable kind="response" params={[
  { name: "type", type: "string", desc: "Fehlertyp: <code>about:blank</code>" },
  { name: "title", type: "string", desc: "HTTP-Statustext, z. B. <code>Bad Request</code>" },
  { name: "status", type: "integer", desc: "HTTP-Status" },
  { name: "detail", type: "string", desc: "Erklärung, was an der Anfrage falsch ist" },
]} />

<Code lang="json" code={`{ "type": "about:blank", "title": "Not Found", "status": 404, "detail": "..." }`} />

## Statuscodes

| Status | Bedeutung | Was tun |
|---|---|---|
| `200` | Erfolg | |
| `304` | Nicht geändert (bei <code>If-None-Match</code>) | Kopie aus dem Cache verwenden |
| `400` | Parameter fehlen oder sind ungültig | <code>detail</code> lesen, Anfrage korrigieren |
| `404` | Produkt unbekannt, falsche Endung, keine Kachel an dieser Stelle oder Zoomstufe | bei Kacheln als leere Kachel behandeln |
| `429` | Limit überschritten | <code>Retry-After</code> Sekunden warten, dann mit wachsendem Abstand erneut |
| `503` | Vorübergehend nicht verfügbar | später erneut versuchen |

## Limits

Limits gelten je IP-Adresse.

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
| `ETag` | Version; mit `If-None-Match` → `304` |

PMTiles-Dateien tragen den Stichtag im Namen und ändern sich nie; sie dürfen unbegrenzt gecacht werden.

## Attribution

JSON-Antworten mit Daten tragen ein Feld `attribution` auf oberster Ebene (Text, ohne HTML) mit der Quelle, die bei Anzeige oder Weitergabe zu nennen ist, etwa `Datenquelle: BEV – DKM, CC BY 4.0` beim Grundstück oder `Datenquelle: GIP.at …` bei Routing, Matrix und Isochrone. Ausnahmen: die Listen `/v1/styles`, `/v1/fonts`, `/v1/sprites` (Arrays); Stile nennen die Quellen in ihren `sources`. Siehe [Attribution](/anleitungen/attribution/).

## Betrieb

- `GET https://api.mapx.at/health` – Lebenszeichen für Monitoring.
- `GET https://api.mapx.at/v1/openapi.json` – OpenAPI-Beschreibung, statische Kopie [openapi.yaml](/openapi.yaml).
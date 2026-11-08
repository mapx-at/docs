# Fehler und Limits

Fehlercodes, Statuscodes, Anfragelimits und Caching aller mapx-APIs.

Jeder Fehler ist ein Dokument `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). Der `type` ist ein URI auf diese Seite: `https://docs.mapx.at/api/errors/#<code>`. Auswertbar sind `code` (oder `type`) und `status`, nie der Text.

<Code lang="json" code={`{
  "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
  "title": "Invalid parameter",
  "status": 400,
  "detail": "types: unknown value 'foo' (allowed: address, street, locality, place, district, region, postcode, poi)",
  "code": "invalid-parameter",
  "param": "types",
  "allowed": ["address", "street", "locality", "place", "district", "region", "postcode", "poi"]
}`} />

## Felder

| Feld | Bedeutung |
|---|---|
| `type` | URI des Fehlertyps (Anker auf dieser Seite) |
| `title` | Kurze Zusammenfassung des Typs |
| `status` | HTTP-Status |
| `detail` | Erklärung zu dieser Anfrage; standardmäßig englisch |
| `code` | Stabiler maschinenlesbarer Code, entspricht dem Anker |
| `param` | Betroffener Parameter (wo sinnvoll) |
| `allowed` | Erlaubte Werte bei Aufzählungen (wo sinnvoll) |

## Sprache

`title` und `detail` sind standardmäßig englisch. Mit `Accept-Language: de` kommen deutsche Texte; `type`, `code`, `param` und `allowed` bleiben gleich. Antworten tragen `Content-Language` und `Vary: Accept-Language`. Das deutsche `detail` ist die allgemeine Beschreibung des Codes, das englische `detail` ist am genauesten.

## Katalog

| Code | Status | Titel |
|---|---|---|
| [`bad-request`](#bad-request) | 400 | Ungültige Anfrage |
| [`missing-parameter`](#missing-parameter) | 400 | Parameter fehlt |
| [`invalid-parameter`](#invalid-parameter) | 400 | Ungültiger Parameter |
| [`invalid-coordinates`](#invalid-coordinates) | 400 | Ungültige Koordinaten |
| [`invalid-body`](#invalid-body) | 400 | Ungültiger Anfragekörper |
| [`unknown-path`](#unknown-path) | 404 | Unbekannter Pfad |
| [`not-found`](#not-found) | 404 | Nicht gefunden |
| [`unknown-style`](#unknown-style) | 404 | Unbekannter Stil |
| [`unknown-tileset`](#unknown-tileset) | 404 | Unbekanntes Tileset |
| [`unknown-font`](#unknown-font) | 404 | Unbekannte Schrift |
| [`unknown-parameter`](#unknown-parameter) | 404 | Unbekannter Parameter |
| [`unknown-id`](#unknown-id) | 404 | Unbekannte Id |
| [`tile-out-of-range`](#tile-out-of-range) | 404 | Kachel außerhalb des Bereichs |
| [`method-not-allowed`](#method-not-allowed) | 405 | Methode nicht erlaubt |
| [`unprocessable`](#unprocessable) | 422 | Anfrage nicht verarbeitbar |
| [`rate-limited`](#rate-limited) | 429 | Rate-Limit überschritten |
| [`internal-error`](#internal-error) | 500 | Interner Fehler |
| [`upstream-error`](#upstream-error) | 502 | Fehler im Backend |
| [`service-unavailable`](#service-unavailable) | 503 | Dienst nicht verfügbar |
| [`service-busy`](#service-busy) | 503 | Dienst ausgelastet |

Token-Fehler (401 `missing-token`, `invalid-token`, `token-expired`; 403 `token-blocked`, `pk-not-allowed`, `origin-missing`, `origin-not-allowed`, `app-not-allowed`, `ip-not-allowed`) stehen mit Ursache und Abhilfe unter [Zugang und Limits](/de/start/access-and-limits/).

### bad-request

**400** · Ungültige Anfrage

Die Anfrage ist ungültig, ohne dass ein spezifischerer Code passt.

Was tun: `detail` lesen und die Anfrage korrigieren.

### missing-parameter

**400** · Parameter fehlt

Ein Pflichtparameter fehlt. `param` nennt ihn.

Was tun: Den in `param` genannten Parameter ergänzen.

### invalid-parameter

**400** · Ungültiger Parameter

Ein Parameter hat einen fehlerhaften Wert, liegt außerhalb des Bereichs oder ist keiner der erlaubten Werte. `param` nennt ihn; bei Aufzählungen listet `allowed` die erlaubten Werte.

Was tun: Einen Wert aus `allowed` oder innerhalb des in `detail` genannten Bereichs verwenden.

### invalid-coordinates

**400** · Ungültige Koordinaten

Eine Koordinate (Pfadsegment oder Parameter) ist nicht lesbar oder liegt außerhalb des gültigen Bereichs. Nicht lesbare Pfadkoordinaten ergeben 400, nicht 404.

Was tun: `lon,lat` in WGS84-Grad senden (Länge -180 bis 180, Breite -90 bis 90).

### invalid-body

**400** · Ungültiger Anfragekörper

Der Körper einer POST-Anfrage ist kein gültiges JSON oder hat nicht die erwartete Form.

Was tun: Das JSON gegen das Schema der OpenAPI-Beschreibung prüfen.

### unknown-path

**404** · Unbekannter Pfad

Diesen Pfad gibt es nicht.

Was tun: Pfad mit der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) vergleichen. Die früheren Aliase `/v1/{tileset}...` und `driving-ev` wurden entfernt.

### not-found

**404** · Nicht gefunden

Die angefragte Ressource (Kachel, Sprite, ...) existiert nicht.

Was tun: Bei Kacheln als leere Kachel behandeln.

### unknown-style

**404** · Unbekannter Stil

Die Stil-Id ist nicht bekannt.

Was tun: Stile mit `/v1/styles` auflisten.

### unknown-tileset

**404** · Unbekanntes Tileset

Der Tileset-Name ist nicht bekannt.

Was tun: Tilesets mit `/v1/tiles` auflisten.

### unknown-font

**404** · Unbekannte Schrift

Der Schriftname ist nicht bekannt.

Was tun: Schriften mit `/v1/fonts` auflisten.

### unknown-parameter

**404** · Unbekannter Parameter

Ein Wetter-Parametername ist nicht bekannt.

Was tun: Einen von temperature, precipitation, wind, humidity, cloud, pressure verwenden.

### unknown-id

**404** · Unbekannte Id

Eine Vorschlags-, Adress- oder Orts-Id ist nicht bekannt (oder nach einem Datenupdate abgelaufen).

Was tun: Neu suchen und die Id aus der frischen Antwort verwenden.

### tile-out-of-range

**404** · Kachel außerhalb des Bereichs

Die Kachel liegt außerhalb des Zoom- oder Kachelbereichs des Tilesets oder Stils.

Was tun: Innerhalb von `minzoom`/`maxzoom` und `bounds` des TileJSON bleiben.

### method-not-allowed

**405** · Methode nicht erlaubt

Erlaubt ist nur GET (und HEAD), POST nur für Batch-Geocoding, Tourenplanung, Adressvalidierung und Verkehrsmeldungen.

Was tun: Die dokumentierte Methode verwenden.

### unprocessable

**422** · Anfrage nicht verarbeitbar

Die Anfrage ist formal gültig, kann aber nicht beantwortet werden, etwa keine Route zwischen den Punkten oder ein Datum außerhalb der Fahrplanperiode.

Was tun: Punkte oder Zeit ändern; `detail` lesen.

### rate-limited

**429** · Rate-Limit überschritten

Zu viele Anfragen in dieser Endpunktgruppe (gezählt je Token, ohne gültigen Token je IP-Adresse). Siehe [Limits](/de/api/errors/#limits).

Was tun: `Retry-After` Sekunden warten, dann mit wachsendem Abstand erneut. Der Header `RateLimit-Policy` jeder Antwort nennt die Zone.

### internal-error

**500** · Interner Fehler

Auf dem Server ist ein unerwarteter Fehler aufgetreten.

Was tun: Einmal wiederholen; bleibt es, die Anfrage melden.

### upstream-error

**502** · Fehler im Backend

Ein Backend-Dienst (Suche, Router) hat nicht oder fehlerhaft geantwortet.

Was tun: Nach einigen Sekunden erneut versuchen.

### service-unavailable

**503** · Dienst nicht verfügbar

Die Daten oder der Dienst sind vorübergehend nicht verfügbar (etwa Wetterdaten noch nicht geladen, Stil wird neu gebaut).

Was tun: Später erneut versuchen.

### service-busy

**503** · Dienst ausgelastet

Der Server ist an seiner Parallelitätsgrenze.

Was tun: `Retry-After` Sekunden warten und erneut versuchen.

Siehe auch: [Fehler und Limits](/de/api/errors/) für Limits und Caching.

## Limits

Die Limits je Zone stehen unter [Zugang und Limits](/de/start/access-and-limits/). Bei `429` und `503` mit wachsendem Abstand wiederholen und `Retry-After` beachten; `400`, `404` und `422` nicht unverändert wiederholen.

## Caching

| Header | Wert |
|---|---|
| `Cache-Control` | `public, max-age=3600` (Kacheln, TileJSON, Höhe, Grundstück), `max-age=300` (Geocoding, Reverse, Autofill-Vorschläge, Address Validation per GET, Tileset-Liste), `max-age=0` (Routing, Matrix, Isochrone, POST-Anfragen) |
| `ETag`, `Last-Modified` | Version; mit `If-None-Match` bzw. `If-Modified-Since` → `304` (Kacheln, Stile, TileJSON, OpenAPI) |
| `Server-Timing` | `router`, `search`, `render`, `total` in Millisekunden |
| `RateLimit-Policy` | Zone und Limit des Endpunkts |

PMTiles-Dateien tragen den Stichtag im Namen und ändern sich nie; sie dürfen unbegrenzt gecacht werden.

## Attribution

JSON-Antworten mit Daten tragen ein Feld `attribution` auf oberster Ebene (Text, ohne HTML) mit der Quelle, die bei Anzeige oder Weitergabe zu nennen ist, etwa `Datenquelle: BEV – DKM, CC BY 4.0` beim Grundstück oder `Datenquelle: GIP.at …` bei Routing, Matrix und Isochrone. Ausnahmen: die Listen `/v1/styles`, `/v1/fonts`, `/v1/sprites` (Arrays); Stile nennen die Quellen in ihren `sources`. Siehe [Attribution](/de/reference/attribution/).
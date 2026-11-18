# Fehlercodes und Rate Limits der API

Source: https://docs.mapx.at/de/api/errors/

> Fehlercodes (problem+json), HTTP-Statuscodes, Rate Limits je Zone und Caching aller mapx-APIs, dazu Retry-After bei 429.

Jeder Fehler ist ein Dokument `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). Der `type` ist ein URI auf diese Seite: `https://docs.mapx.at/api/errors/#<code>`. Auswertbar sind `code` (oder `type`) und `status`, nie der Text.

```json
{
  "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
  "title": "Invalid parameter",
  "status": 400,
  "detail": "types: unknown value 'foo' (allowed: address, street, locality, place, district, region, postcode, poi, parcel, road_ref)",
  "code": "invalid-parameter",
  "param": "types",
  "allowed": ["address", "street", "locality", "place", "district", "region", "postcode", "poi", "parcel", "road_ref"]
}
```

## Felder

| Feld | Bedeutung |
|---|---|
| `type` | URI des Fehlertyps (Anker auf dieser Seite) |
| `title` | Kurze Zusammenfassung des Typs |
| `status` | HTTP-Status |
| `detail` | Erklärung zu dieser Anfrage; standardmäßig englisch. Fehlt bei Anmelde- und Token-Fehlern (siehe unten) |
| `code` | Stabiler maschinenlesbarer Code, entspricht dem Anker |
| `param` | Betroffener Parameter (wo sinnvoll) |
| `allowed` | Erlaubte Werte bei Aufzählungen (wo sinnvoll) |

## Anmeldefehler

Jede Anfrage braucht einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Anmeldefehler antworten mit `401` oder `403` als `application/problem+json`; der Code steht zusätzlich im Antwort-Header `X-Mapx-Reason` (aus Browser-Skripten lesbar). Anfragen ohne Token bekommen `401` `missing-token`.

Diese Antworten kommen von der Token-Prüfung vor der API und sind knapper als die übrigen: `title` ist allgemein (`Unauthorized` bei 401, `Forbidden` bei 403, `Too many requests` bei 429 durch das Minutenlimit des Tokens), immer englisch, und es gibt kein `detail`. Auswerten lässt sich nur `code` bzw. `X-Mapx-Reason`.

```json
{"type": "https://docs.mapx.at/api/errors/#missing-token", "title": "Unauthorized", "status": 401, "code": "missing-token"}
```

```bash
# mit einem ungültigen oder unbekannten Token (hier dem Platzhalter selbst) kommt 401 invalid-token
curl -si -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk" | head -12
```

## Sprache

`title` und `detail` sind standardmäßig englisch. Außer bei Anmeldefehlern kommen mit `Accept-Language: de` kommen deutsche Texte; `type`, `code`, `param` und `allowed` bleiben gleich. Antworten tragen `Content-Language` und `Vary: Accept-Language`. Das deutsche `detail` ist die allgemeine Beschreibung des Codes, das englische `detail` ist am genauesten.

## Katalog

| Code | Status | Titel |
|---|---|---|
| [`bad-request`](#bad-request) | 400 | Ungültige Anfrage |
| [`missing-parameter`](#missing-parameter) | 400 | Parameter fehlt |
| [`invalid-parameter`](#invalid-parameter) | 400 | Ungültiger Parameter |
| [`invalid-coordinates`](#invalid-coordinates) | 400 | Ungültige Koordinaten |
| [`invalid-body`](#invalid-body) | 400 | Ungültiger Anfragekörper |
| [`missing-token`](#missing-token) | 401 | Unauthorized (Token fehlt) |
| [`invalid-token`](#invalid-token) | 401 | Unauthorized (Token ungültig) |
| [`token-expired`](#token-expired) | 401 | Unauthorized (Token abgelaufen) |
| [`token-blocked`](#token-blocked) | 403 | Forbidden (Token gesperrt) |
| [`origin-not-allowed`](#origin-not-allowed) | 403 | Forbidden (Website nicht erlaubt) |
| [`origin-missing`](#origin-missing) | 403 | Forbidden (Herkunft fehlt) |
| [`pk-not-allowed`](#pk-not-allowed) | 403 | Forbidden (öffentlicher Token hier nicht erlaubt) |
| [`app-not-allowed`](#app-not-allowed) | 403 | Forbidden (App nicht erlaubt) |
| [`ip-not-allowed`](#ip-not-allowed) | 403 | Forbidden (IP-Adresse nicht erlaubt) |
| [`mcp-token-only`](#mcp-token-only) | 403 | Forbidden (MCP-Token nur am MCP-Server gültig) |
| [`mcp-token-required`](#mcp-token-required) | 403 | MCP-Token erforderlich |
| [`invalid-path`](#invalid-path) | 403 | Forbidden (ungültiger Anfragepfad) |
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
| [`rate-limited`](#rate-limited) | 429 | Rate-Limit überschritten bzw. Too many requests |
| [`internal-error`](#internal-error) | 500 | Interner Fehler |
| [`upstream-error`](#upstream-error) | 502 | Fehler im Backend |
| [`service-unavailable`](#service-unavailable) | 503 | Dienst nicht verfügbar |
| [`service-busy`](#service-busy) | 503 | Dienst ausgelastet |
| [`cache-unavailable`](#cache-unavailable) | 503 | Authorization backend unavailable |

In Klammern steht bei Anmeldefehlern die Bedeutung; die API sendet nur den allgemeinen Titel davor.

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

### missing-token

**401** · Token fehlt

Die Anfrage enthält keinen Token.

Was tun: Unter [account.mapx.at](https://account.mapx.at/) einen Token anlegen und als `Authorization: Bearer YOUR_TOKEN` oder `?access_token=YOUR_TOKEN` senden. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

### invalid-token

**401** · Token ungültig

Der Token ist fehlerhaft oder unbekannt (auch nach Löschen oder Rotieren).

Was tun: Den Token erneut aus dem Konto kopieren, auf Leerzeichen oder Zeilenumbrüche achten und den aktuellen Token verwenden. Tokens beginnen mit `mx_pk_`, `mx_sk_` oder (nur für den MCP-Server) `mx_mcp_`.

### token-expired

**401** · Token abgelaufen

Der Token ist über sein Ablaufdatum hinaus.

Was tun: Im Konto einen Token anlegen oder rotieren.

### token-blocked

**403** · Token gesperrt

Der Token wurde im Konto gesperrt.

Was tun: Im Konto entsperren oder einen anderen Token verwenden.

### origin-not-allowed

**403** · Website nicht erlaubt

Ein öffentlicher Token (`mx_pk_`) wurde von einer Website verwendet, deren `Origin` (bzw. `Referer`-Host) nicht in den erlaubten Websites des Tokens steht.

Was tun: Die Website am Token eintragen (für die Entwicklung auch `localhost`) oder die API vom eigenen Server mit einem geheimen Token aufrufen.

### origin-missing

**403** · Herkunft fehlt

Ein auf Websites beschränkter öffentlicher Token wurde ohne `Origin` und ohne `Referer` verwendet; die Website lässt sich nicht prüfen.

Was tun: Aus einer Browserseite aufrufen oder auf dem Server einen geheimen Token verwenden.

### pk-not-allowed

**403** · Öffentlicher Token hier nicht erlaubt

Ein öffentlicher Token erlaubt nur lesenden Zugriff (`GET`, `HEAD`) auf die öffentlichen Produkte. Batch-Geocoding (`POST /v1/geocode/batch`), Tourenplanung (`POST /v1/optimization`) und `corridor-graph` gehen nur mit einem geheimen Token.

Was tun: Diese Anfrage von einem Server mit einem geheimen Token (`mx_sk_`) senden.

### app-not-allowed

**403** · App nicht erlaubt

Ein an eine Android-App gebundener öffentlicher Token wurde mit einem Paketnamen oder Signaturzertifikat verwendet, das nicht passt.

Was tun: Die Header `X-Mapx-Android-Package` und `X-Mapx-Android-Cert` der registrierten App senden oder die Bindung im Konto korrigieren.

### ip-not-allowed

**403** · IP-Adresse nicht erlaubt

Der geheime Token ist auf IP-Bereiche beschränkt, die Anfrage kommt von einer anderen Adresse.

Was tun: Die Adresse (CIDR) am Token eintragen oder von einem erlaubten Host senden.

### mcp-token-only

**403** · MCP-Token nur am MCP-Server gültig

Ein Token vom Typ KI-Assistent (`mx_mcp_`) wurde für einen direkten API-Aufruf verwendet. Er gilt nur am [MCP-Server](https://docs.mapx.at/de/guides/mcp.md).

Was tun: Für direkte API-Aufrufe einen geheimen Token (`mx_sk_`) verwenden.

### mcp-token-required

**403** · MCP-Token erforderlich

Der MCP-Server `https://mcp.mapx.at/mcp` nimmt nur Tokens vom Typ KI-Assistent (`mx_mcp_`) an; gesendet wurde ein öffentlicher oder geheimer Token.

Was tun: Unter [account.mapx.at](https://account.mapx.at/tokens/new?kind=mcp) einen Token vom Typ KI-Assistent (MCP) anlegen.

### invalid-path

**403** · Ungültiger Anfragepfad

Der Anfragepfad hat eine ungültige Prozent-Kodierung, ein NUL-Byte oder verlässt mit `..` die Wurzel.

Was tun: Einen normalen Pfad aus der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) senden.

### unknown-path

**404** · Unbekannter Pfad

Diesen Pfad gibt es nicht.

Was tun: Pfad mit der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) vergleichen. Ein unbekanntes Routing-Profil wie das frühere `driving-ev` antwortet mit `400` `invalid-parameter`.

### not-found

**404** · Nicht gefunden

Die angefragte Ressource (Kachel, Sprite, ...) existiert nicht.

Was tun: Pfad prüfen. Kacheln im Zoombereich ohne Daten antworten mit `204`, nicht mit diesem Fehler.

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

Zu viele Anfragen. `429` hat zwei Ursachen: das Limit je IP-Adresse und Zone (Endpunktgruppe; Antwort mit `title` `Rate limit exceeded` und `detail`) oder das Minutenlimit je Token (Antwort der Token-Prüfung mit `title` `Too many requests`, ohne `detail`). Beide tragen `Retry-After`. Siehe [Limits](https://docs.mapx.at/de/api/errors.md#limits).

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

### cache-unavailable

**503** · Authorization backend unavailable

Der Speicher für die Token-Prüfung ist kurz nicht erreichbar und der Token ist nicht im lokalen Zwischenspeicher. Die Anfrage wird zur Sicherheit abgelehnt.

Was tun: Nach einigen Sekunden erneut versuchen.

Siehe auch: [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md) für Limits und Caching.

## Limits

Die Limits je IP-Adresse und Zone und das Minutenlimit je Token stehen unter [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Bei `429` und `503` mit wachsendem Abstand wiederholen und `Retry-After` beachten; `400`, `401`, `403`, `404` und `422` nicht unverändert wiederholen.

## Caching

| Header | Wert |
|---|---|
| `Cache-Control` | `public, max-age=3600` (Kacheln, TileJSON, Höhe, Grundstück), `max-age=300` (Geocoding, Reverse, Autofill-Vorschläge, Address Validation per GET, Tileset-Liste), `max-age=0` (Routing, Matrix, Isochrone, POST-Anfragen) |
| `ETag`, `Last-Modified` | Version; mit `If-None-Match` bzw. `If-Modified-Since` → `304` (Kacheln, Stile, TileJSON, OpenAPI) |
| `Server-Timing` | `router`, `search`, `render`, `total` in Millisekunden |
| `RateLimit-Policy` | Zone und Limit des Endpunkts |

PMTiles-Dateien tragen den Stichtag im Namen und ändern sich nie; sie dürfen unbegrenzt gecacht werden.

## Attribution

JSON-Antworten mit Daten tragen ein Feld `attribution` auf oberster Ebene (Text, ohne HTML) mit der Quelle, die bei Anzeige oder Weitergabe zu nennen ist, etwa `Datenquelle: BEV – DKM, CC BY 4.0` beim Grundstück oder `Datenquelle: GIP.at …` bei Routing, Matrix und Isochrone. Ausnahmen: die Listen `/v1/styles`, `/v1/fonts`, `/v1/sprites` (Arrays); Stile nennen die Quellen in ihren `sources`. Siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

# Fehlercodes und Rate Limits der API

Source: https://docs.mapx.at/de/api/errors/

> Fehlercodes (problem+json), HTTP-Statuscodes, Rate Limits je Zone und Caching aller mapx-APIs, dazu Retry-After bei 429.

Jeder Fehler ist ein Dokument `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)). Der `type` ist ein URI auf diese Seite: `https://docs.mapx.at/api/errors/#<code>`. Auswertbar sind `code` (oder `type`) und `status`, nie der Text.

```json
{
  "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
  "title": "Invalid parameter",
  "status": 400,
  "detail": "types: unknown value 'foo' (allowed: address, street, locality, place, district, region, postcode, poi, road_ref)",
  "code": "invalid-parameter",
  "param": "types",
  "allowed": ["address", "street", "locality", "place", "district", "region", "postcode", "poi", "road_ref"]
}
```

## Felder

| Feld | Bedeutung |
|---|---|
| `type` | URI des Fehlertyps (Anker auf dieser Seite) |
| `title` | Kurze Zusammenfassung des Typs |
| `status` | HTTP-Status |
| `detail` | Erklärung zu dieser Anfrage; standardmäßig englisch |
| `code` | Stabiler maschinenlesbarer Code, entspricht dem Anker |
| `param` | Betroffener Parameter (wo sinnvoll); bei Pfadteilen ihr Name in der OpenAPI-Beschreibung (`coordinates`, `coordinate`, `category`), bei Feldern eines JSON-Körpers der Feldpfad (`vehicles[0].start`) |
| `allowed` | Erlaubte Werte bei Aufzählungen (wo sinnvoll) |

## Anmeldefehler

Jede Anfrage braucht einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Anmeldefehler antworten mit `401` oder `403` als `application/problem+json`; der Code steht zusätzlich im Antwort-Header `X-Mapx-Reason` (aus Browser-Skripten lesbar). Anfragen ohne Token bekommen `401` `missing-token`.

Diese Antworten kommen von der Token-Prüfung im Gateway vor der API. Sie haben dieselbe Form wie alle anderen Fehler: `title` und `detail` je Grund (etwa `Access token required` bei `missing-token`, `Invalid access token` bei `invalid-token`), immer englisch. Ausgewertet wird `code` bzw. `X-Mapx-Reason`.

```json
{"type": "https://docs.mapx.at/api/errors/#missing-token", "title": "Access token required", "status": 401, "detail": "Send the token as 'Authorization: Bearer <token>' or '?access_token=<token>'.", "code": "missing-token"}
```

Auch die übrigen Fehler, die das Gateway selbst erzeugt (`404` `unknown-path`, `405`, `413`, `414`, `429`, `500`, `502`, `503`, `504`), kommen als `application/problem+json` mit `type`, `title`, `detail` und `code`. Die Methode prüft das Gateway vor dem Token: eine nicht erlaubte Methode ergibt `405` auch dann, wenn der Token fehlt.

```bash
# mit einem ungültigen oder unbekannten Token (hier dem Platzhalter selbst) kommt 401 invalid-token
curl -si -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk" | head -12
```

## Sprache

`detail` ist immer englisch und nennt die genaue Grenze bzw. den Wert (zum Beispiel `limit: 1 to 10`). Mit `Accept-Language: de` ist `title` deutsch, `detail` bleibt gleich, es geht also nichts verloren; Anmeldefehler und die übrigen Fehler des Gateways (`405`, `413`, `414`, `429`, `5xx`) sind immer englisch (`unknown-path` gibt es auf Deutsch). `type`, `code`, `param` und `allowed` hängen nie von der Sprache ab. Antworten tragen `Content-Language` und `Vary: Accept-Language`.

## Katalog

| Code | Status | Titel |
|---|---|---|
| [`bad-request`](#bad-request) | 400 | Ungültige Anfrage |
| [`missing-parameter`](#missing-parameter) | 400 | Parameter fehlt |
| [`invalid-parameter`](#invalid-parameter) | 400 | Ungültiger Parameter |
| [`invalid-coordinates`](#invalid-coordinates) | 400 | Ungültige Koordinaten |
| [`invalid-body`](#invalid-body) | 400 | Ungültiger Anfragekörper |
| [`missing-token`](#missing-token) | 401 | Access token required |
| [`invalid-token`](#invalid-token) | 401 | Invalid access token |
| [`token-expired`](#token-expired) | 401 | Access token expired |
| [`token-blocked`](#token-blocked) | 403 | Access token blocked |
| [`origin-not-allowed`](#origin-not-allowed) | 403 | Origin not allowed |
| [`origin-missing`](#origin-missing) | 403 | Origin missing |
| [`pk-not-allowed`](#pk-not-allowed) | 403 | Public token not allowed here |
| [`app-not-allowed`](#app-not-allowed) | 403 | App not allowed |
| [`ip-not-allowed`](#ip-not-allowed) | 403 | IP not allowed |
| [`mcp-token-only`](#mcp-token-only) | 403 | MCP token only valid at the MCP server |
| [`mcp-token-required`](#mcp-token-required) | 403 | MCP token required |
| [`invalid-path`](#invalid-path) | 403 | Invalid request path |
| [`forbidden`](#forbidden) | 403 | Forbidden |
| [`range-required`](#range-required) | 403 | Range erforderlich |
| [`unknown-path`](#unknown-path) | 404 | Unbekannter Pfad |
| [`not-found`](#not-found) | 404 | Nicht gefunden |
| [`unknown-style`](#unknown-style) | 404 | Unbekannter Stil |
| [`unknown-tileset`](#unknown-tileset) | 404 | Unbekanntes Tileset |
| [`unknown-font`](#unknown-font) | 404 | Unbekannte Schrift |
| [`unknown-parameter`](#unknown-parameter) | 400 | Unbekannter Parameter |
| [`unknown-layer`](#unknown-layer) | 404 | Unbekannte Ebene |
| [`unknown-id`](#unknown-id) | 404 | Unbekannte Id |
| [`no-road-nearby`](#no-road-nearby) | 404 | Keine Straße in der Nähe |
| [`tile-out-of-range`](#tile-out-of-range) | 404 | Kachel außerhalb des Bereichs |
| [`outside-coverage`](#outside-coverage) | 404 | Außerhalb des Datengebiets |
| [`method-not-allowed`](#method-not-allowed) | 405 | Methode nicht erlaubt |
| [`payload-too-large`](#payload-too-large) | 413 | Payload too large |
| [`uri-too-long`](#uri-too-long) | 414 | URI too long |
| [`unprocessable`](#unprocessable) | 422 | Anfrage nicht verarbeitbar |
| [`rate-limited`](#rate-limited) | 429 | Rate limit exceeded |
| [`quota-exceeded`](#quota-exceeded) | 429 | Quota exceeded |
| [`internal-error`](#internal-error) | 500 | Interner Fehler |
| [`upstream-error`](#upstream-error) | 502 | Fehler im Backend |
| [`service-unavailable`](#service-unavailable) | 503 | Dienst nicht verfügbar |
| [`service-busy`](#service-busy) | 503 | Dienst ausgelastet |
| [`cache-unavailable`](#cache-unavailable) | 503 | Authorization backend unavailable |
| [`upstream-timeout`](#upstream-timeout) | 504 | Upstream timeout |

Titel der Anmeldefehler und der übrigen Gateway-Fehler stehen so, wie sie gesendet werden (englisch).

### bad-request

**400** · Ungültige Anfrage

Die Anfrage ist ungültig, ohne dass ein spezifischerer Code passt.

Was tun: `detail` lesen und die Anfrage korrigieren.

### missing-parameter

**400** · Parameter fehlt

Ein Pflichtparameter fehlt. `param` nennt ihn. Ist einer von zwei Parametern Pflicht (`bbox` oder `near` bei `/v1/stops`, `bbox` oder `corridor` bei `/v1/speedlimits`) und fehlen beide, nennt `param` den ersten; sind beide gesetzt, kommt `invalid-parameter` mit dem zweiten.

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

**401** · Access token required

Die Anfrage enthält keinen Token.

Was tun: Unter [account.mapx.at](https://account.mapx.at/) einen Token anlegen und als `Authorization: Bearer YOUR_TOKEN` oder `?access_token=YOUR_TOKEN` senden. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

### invalid-token

**401** · Invalid access token

Der Token ist fehlerhaft oder unbekannt (auch nach Löschen oder Rotieren).

Was tun: Den Token erneut aus dem Konto kopieren, auf Leerzeichen oder Zeilenumbrüche achten und den aktuellen Token verwenden. Tokens beginnen mit `mx_pk_`, `mx_sk_` oder (nur für den MCP-Server) `mx_mcp_`.

### token-expired

**401** · Access token expired

Der Token ist über sein Ablaufdatum hinaus.

Was tun: Im Konto einen Token anlegen oder rotieren.

### token-blocked

**403** · Access token blocked

Der Token wurde im Konto gesperrt.

Was tun: Im Konto entsperren oder einen anderen Token verwenden.

### origin-not-allowed

**403** · Origin not allowed

Ein öffentlicher Token (`mx_pk_`) wurde von einer Website verwendet, deren `Origin` (bzw. `Referer`-Host) nicht in den erlaubten Websites des Tokens steht.

Was tun: Die Website am Token eintragen (für die Entwicklung auch `localhost`) oder die API vom eigenen Server mit einem geheimen Token aufrufen.

### origin-missing

**403** · Origin missing

Ein auf Websites beschränkter öffentlicher Token wurde ohne `Origin` und ohne `Referer` verwendet; die Website lässt sich nicht prüfen.

Was tun: Aus einer Browserseite aufrufen oder auf dem Server einen geheimen Token verwenden.

### pk-not-allowed

**403** · Public token not allowed here

Ein öffentlicher Token erlaubt nur lesenden Zugriff (`GET`, `HEAD`) auf die öffentlichen Produkte. Batch-Geocoding (`POST /v1/geocode/batch`), Tourenplanung (`POST /v1/optimization`), Adressprüfung per `POST /v1/address/validate`, Verkehrsmeldungen per `POST /v1/traffic/incidents` und `GET /v1/corridor-graph` gehen nur mit einem geheimen Token.

Was tun: Diese Anfrage von einem Server mit einem geheimen Token (`mx_sk_`) senden.

### app-not-allowed

**403** · App not allowed

Ein an eine Android-App gebundener öffentlicher Token wurde mit einem Paketnamen oder Signaturzertifikat verwendet, das nicht passt.

Was tun: Die Header `X-Mapx-Android-Package` und `X-Mapx-Android-Cert` der registrierten App senden oder die Bindung im Konto korrigieren.

### ip-not-allowed

**403** · IP not allowed

Der geheime Token ist auf IP-Bereiche beschränkt, die Anfrage kommt von einer anderen Adresse.

Was tun: Die Adresse (CIDR) am Token eintragen oder von einem erlaubten Host senden.

### mcp-token-only

**403** · MCP token only valid at the MCP server

Ein Token vom Typ KI-Assistent (`mx_mcp_`) wurde für einen direkten API-Aufruf verwendet. Er gilt nur am [MCP-Server](https://docs.mapx.at/de/guides/mcp.md).

Was tun: Für direkte API-Aufrufe einen geheimen Token (`mx_sk_`) verwenden.

### mcp-token-required

**403** · MCP token required

Der MCP-Server `https://mcp.mapx.at/mcp` nimmt nur Tokens vom Typ KI-Assistent (`mx_mcp_`) an; gesendet wurde ein öffentlicher oder geheimer Token.

Was tun: Unter [account.mapx.at](https://account.mapx.at/tokens/new?kind=mcp) einen Token vom Typ KI-Assistent (MCP) anlegen.

### invalid-path

**403** · Invalid request path

Der Anfragepfad hat eine ungültige Prozent-Kodierung, ein NUL-Byte oder verlässt mit `..` die Wurzel.

Was tun: Einen normalen Pfad aus der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) senden.

### forbidden

**403** · Forbidden

Der Zugriff ist ohne spezifischeren Grund abgelehnt, etwa ein Pfad, der am Gateway gesperrt ist.

Was tun: Pfad und Host mit der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json) vergleichen.

### range-required

**403** · Range erforderlich

`tiles.mapx.at` liefert PMTiles-Dateien nur stückweise aus: Ein `GET` auf eine `.pmtiles`-Datei braucht genau einen geschlossenen `Range`-Header (`bytes=<start>-<ende>`). Anfragen ohne `Range`, mit offener (`bytes=100-`), Suffix- (`bytes=-500`) oder Mehrfach-Range werden abgelehnt. MapLibre mit pmtiles.js und die mapx-Styles senden von sich aus passende Ranges. Das Gateway prüft das wie die Methode vor dem Token.

Was tun: Karten über den [Style](https://docs.mapx.at/de/api/styles.md), [TileJSON oder `{z}/{x}/{y}`-Kacheln](https://docs.mapx.at/de/api/tiles.md) laden. Ganze Dateien gibt es nicht zum Herunterladen.

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

**400** · Unbekannter Parameter

Der Endpunkt kennt einen Query-Parameter oder ein Feld des JSON-Körpers nicht (oft ein Tippfehler, etwa `limt` statt `limit`). `param` nennt ihn, `detail` zählt die bekannten Parameter des Endpunkts auf. `access_token` ist überall erlaubt.

Was tun: Den Namen korrigieren oder den Parameter weglassen; die Parameter je Endpunkt stehen in der [OpenAPI-Beschreibung](https://api.mapx.at/v1/openapi.json).

### unknown-layer

**404** · Unbekannte Ebene

Eine Wetter- oder Luftgüte-Ebene (Pfad von `/v1/weather/tiles/…`, `/v1/air-quality/tiles/…`) ist nicht bekannt.

Was tun: Einen von temperature, precipitation, wind, humidity, cloud, pressure, aqi, no2, o3, pm10, pm25, dust, snowlimit, sunshine, radiation, cape, snowfall, rr_nowcast, pt_nowcast verwenden.

### unknown-id

**404** · Unbekannte Id

Eine Vorschlags-, Adress- oder Orts-Id ist nicht bekannt (oder nach einem Datenupdate abgelaufen).

Was tun: Neu suchen und die Id aus der frischen Antwort verwenden. Bei `/v1/departures` mit unbekannter Stations-Id nennt `param` den Parameter `stop`.

### no-road-nearby

**404** · Keine Straße in der Nähe

Im Suchradius um einen Punkt liegt keine für das Profil befahrbare Straße (Routing, Isochrone, Matrix: Standard 1000 m, je Punkt über `radiuses`), bzw. entlang der Linie von `/v1/speedlimits` (`corridor`) und `/v1/corridor-graph` (`polyline`) liegt im Puffer keine. `param` nennt den Parameter (`coordinates`, `coordinate`, `corridor`, `polyline`), `detail` den Punkt (`start`, `destination`) und den Radius.

Was tun: Den Punkt näher an eine Straße legen oder den Radius (`radiuses`, `buffer`) vergrößern; Koordinaten in der Reihenfolge `lon,lat` prüfen.

### tile-out-of-range

**404** · Kachel außerhalb des Bereichs

Die Kachel liegt außerhalb des Zoom- oder Kachelbereichs des Tilesets oder Stils.

Was tun: Innerhalb von `minzoom`/`maxzoom` und `bounds` des TileJSON bleiben.

### outside-coverage

**404** · Außerhalb des Datengebiets

Der Punkt liegt außerhalb des Gebiets, für das Daten vorliegen (Wetter, Nowcast, Luftgüte und Klima decken Österreich und Umgebung ab). `detail` nennt die Daten und ergänzt „(lon and lat swapped?)“, wenn die vertauschten Koordinaten im Gebiet lägen.

Was tun: Einen Punkt in Österreich als `lon,lat` senden (Länge etwa 9,5 bis 17,2, Breite etwa 46,4 bis 49,0).

### method-not-allowed

**405** · Methode nicht erlaubt

Erlaubt ist nur GET (und HEAD), POST nur für Batch-Geocoding, Tourenplanung, Adressvalidierung und Verkehrsmeldungen. Der Header `Allow` nennt die erlaubten Methoden (z. B. `POST /v1/geocode` → `405`, `Allow: GET, HEAD`). Das Gateway prüft die Methode vor dem Token, daher kommt `405` auch, wenn zusätzlich der Token fehlt.

Was tun: Die Methode aus `Allow` verwenden.

### payload-too-large

**413** · Payload too large

Der Anfragekörper ist größer, als der Endpunkt annimmt: Batch-Geocoding 512 KB, Tourenplanung 256 KB, Verkehrsmeldungen 64 KB, alle übrigen POST-Endpunkte (etwa Adressvalidierung) 16 KB; der MCP-Server 256 KB.

Was tun: Die Anfrage aufteilen oder verkleinern.

### uri-too-long

**414** · URI too long

Die Anfrage-URL ist länger als etwa 8 KB, z. B. sehr viele `points` bei `/v1/elevation`.

Was tun: Weniger Werte je Anfrage senden oder, wo der Endpunkt es anbietet, POST mit JSON-Körper verwenden.

### unprocessable

**422** · Anfrage nicht verarbeitbar

Die Anfrage ist formal gültig, kann aber nicht beantwortet werden, etwa keine Route zwischen den Punkten, kein Map Matching, keine Reihenfolge bei der Optimierung, keine Haltestelle in Reichweite oder ein Datum außerhalb der Fahrplanperiode.

Was tun: Punkte oder Zeit ändern; `detail` lesen.

### rate-limited

**429** · Rate limit exceeded

Zu viele Anfragen. `429` hat zwei Ursachen: das Limit je IP-Adresse und Zone (Endpunktgruppe) oder das Minutenlimit je Token. Beide antworten mit `title` `Rate limit exceeded`, `detail` und `Retry-After`. Siehe [Limits](https://docs.mapx.at/de/api/errors.md#limits).

Teure Gruppen haben zusätzlich ein Gesamtlimit für alle Clients zusammen; wird es erreicht, kommt dieselbe Antwort.

Was tun: `Retry-After` Sekunden warten, dann mit wachsendem Abstand erneut. Der Header `RateLimit-Policy` jeder Antwort (vom Gateway aus seiner Zonentabelle gesetzt) nennt die Zone.

In der Nutzungsansicht des Kontos steht das Minutenlimit je Token als `rate-limited`, das Limit einer Zone als `rate-limited:<Zone>` (z. B. `rate-limited:search`, `rate-limited:connections` bei zu vielen gleichzeitigen Verbindungen), jeweils beim Token der Anfrage.

### quota-exceeded

**429** · Quota exceeded

Das Tages- oder Monatslimit des Kontos ist ausgeschöpft (alle Tokens des Kontos zusammen, Tage und Monate in UTC). Derzeit sind beide Limits unbegrenzt, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md#rate-limits).

Was tun: `Retry-After` nennt die Sekunden bis zum nächsten Tag bzw. Monat (UTC); bis dahin wiederholen hilft nicht. Den Verbrauch zeigt das Konto unter Nutzung.

### internal-error

**500** · Interner Fehler

Auf dem Server oder im Gateway ist ein unerwarteter Fehler aufgetreten.

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

### upstream-timeout

**504** · Upstream timeout

Der Dienst hinter dem Gateway hat nicht rechtzeitig geantwortet.

Was tun: Nach einigen Sekunden erneut versuchen, bei großen Anfragen kleiner aufteilen.

Siehe auch: [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md) für Limits und Caching.

## Limits

Die Limits je IP-Adresse und Zone und das Minutenlimit je Token stehen unter [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Bei `429` und `503` mit wachsendem Abstand wiederholen und `Retry-After` beachten; `400`, `401`, `403`, `404` und `422` nicht unverändert wiederholen.

## Caching

| Header | Wert |
|---|---|
| `Cache-Control` | `private` für jede Antwort auf eine Anfrage mit Token (nur Browser-Cache, keine gemeinsamen Caches), mit `max-age=3600` (tiles.mapx.at, TileJSON, Höhe), `max-age=86400` (Kacheln über api.mapx.at), `max-age=60` (Search-Box-Vorschläge, Verkehr), `max-age=30` (Abfahrten), `max-age=300` (Geocoding, Reverse, Autofill-Vorschläge, Address Validation per GET, Kachelsatz-Liste), `max-age=0` (Routing, Matrix, Isochrone, POST-Anfragen); `/health` `no-store` |
| `ETag` | bei jeder `GET`-Antwort; mit `If-None-Match` → `304` ohne Inhalt. Kacheln, Stile, TileJSON und OpenAPI: Dateistand (dazu `Last-Modified`, `If-Modified-Since`), alle übrigen: Prüfsumme des Inhalts |
| `Server-Timing` | `router`, `search`, `render`, `total` in Millisekunden |
| `RateLimit-Policy` | Zone und Limit des Endpunkts (vom Gateway gesetzt) |

PMTiles-Dateien tragen den Stichtag im Namen und ändern sich nie; sie dürfen unbegrenzt gecacht werden.

## Attribution

JSON-Antworten mit Daten tragen ein Feld `attribution` auf oberster Ebene (Text, ohne HTML) mit der Quelle, die bei Anzeige oder Weitergabe zu nennen ist. Sie beginnt immer mit `mapx.at; `, etwa `mapx.at; Datenquelle: GeoSphere Austria, CC BY 4.0 (…)` beim Wetter oder `mapx.at; Datenquelle: GIP.at …` bei Routing, Matrix und Isochrone. Ausnahmen: die Listen `/v1/styles`, `/v1/fonts`, `/v1/sprites` (Arrays); Stile nennen die Quellen in ihren `sources`, `mapx.at` als erste Quelle `mapx`. Siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

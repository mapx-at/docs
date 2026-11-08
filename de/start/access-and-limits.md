# Zugang und Limits

Kostenloser API-Token, öffentliche und geheime Tokens, Übergabe, Rate Limits pro Zone, CORS, Caching und Attribution.

## Token holen

Alle Endpunkte unter `https://api.mapx.at/v1/`, `https://routing.mapx.at/`, `https://search.mapx.at/` und alle Tiles unter `https://tiles.mapx.at/` brauchen einen Token. Er ist kostenlos, es gibt keine nutzungsabhängige Verrechnung.

1. [account.mapx.at](https://account.mapx.at/) öffnen und mit der E-Mail-Adresse registrieren (ein 8-stelliger Code kommt per Mail) oder mit Google bzw. Microsoft anmelden. Der Kontaktaufnahme zu Dienst und Konto (Sicherheit, Änderungen, Abschaltung) ist zuzustimmen. Werbung und Newsletter sind eine eigene, freiwillige Checkbox.
2. Unter **Tokens** beliebig viele benannte Tokens anlegen. Der Token wird nur einmal angezeigt, dann kopieren. Tokens lassen sich jederzeit sperren, rotieren oder löschen, Löschen wirkt sofort.
3. Den Token bei jeder Anfrage mitschicken (siehe unten). Die Seite **Nutzung** zeigt Anfragen je Token, Produkt und Tag, auch abgelehnte mit Grund.

Die OpenAPI-Beschreibung `https://api.mapx.at/v1/openapi.json` und CORS-Preflights (`OPTIONS`) funktionieren ohne Token.

## Zwei Token-Arten

| Art | Präfix | Einsatz | Einschränkung |
|---|---|---|---|
| Öffentlich | `mx_pk_…` | Webseiten und Apps. Der Token ist im Quelltext bzw. in der APK sichtbar und wirkt deshalb nur dort, wo es erlaubt ist. | Nur lesende Produkte, nur `GET`/`HEAD`. Gebunden an **erlaubte Origins** (etwa `https://example.com`, `*.example.com`) oder an eine **Android-App** (Paketname plus SHA-256 des Signaturzertifikats). |
| Geheim | `mx_sk_…` | Server, Skripte und CI. Nie in eine Webseite oder App einbauen. | Keine Origin-Bindung; optional an IP-Bereiche (CIDR) gebunden. |

Ein öffentlicher Token wird gegen den Header `Origin` geprüft (bei Kachel- und Bildanfragen ohne Origin gegen den Host des `Referer`). Anfragen von anderen Seiten bekommen `403` mit dem Grund `origin-not-allowed`. `localhost` nur während der Entwicklung als Origin eintragen.

## Token mitschicken

Beides funktioniert: den Header verwenden, wo es geht, den Query-Parameter dort, wo keine Header möglich sind (Bilder, MapLibre-Kachel-URLs).

```bash
# Header
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=Melk&limit=1" | jq

# Query-Parameter
curl -s "https://api.mapx.at/v1/geocode?q=Melk&limit=1&access_token=DEIN_TOKEN" | jq
```

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Melk", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
```

**Karten mit MapLibre GL JS.** Am einfachsten mit `transformRequest`, das den Token an jede Anfrage an `mapx.at` hängt, nie an andere Hosts:

```js
const TOKEN = "DEIN_TOKEN"; // öffentlicher Token mx_pk_…
const map = new maplibregl.Map({
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets",
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
});
```

Wird ein Style mit `?access_token=…` angefordert, schreibt die API den Token in alle `mapx.at`-URLs des Styles (Tiles, Glyphs, Sprites). Clients ohne `transformRequest` kommen dann mit `style: "https://api.mapx.at/v1/styles/streets?access_token=DEIN_TOKEN"` aus. Diese Antwort ist `private` und darf nicht geteilt werden.

**Mit dem SDK.** `mapx.map({ accessToken: "DEIN_TOKEN", … })` oder einmal `mapx.setAccessToken("DEIN_TOKEN")`; Helfer, Styles, Tiles und `staticImageUrl()` tragen den Token dann mit. Siehe [SDK](/de/guides/sdk/).

**PMTiles-Dateien.** Token an die Datei-URL hängen: `pmtiles://https://tiles.mapx.at/<datei>.pmtiles?access_token=DEIN_TOKEN`.

**Android.** `Authorization: Bearer <öffentlicher Token>` plus `X-Mapx-Android-Package` (Paketname) und `X-Mapx-Android-Cert` (SHA-256 des Signaturzertifikats, zur Laufzeit per `PackageManager` gelesen).

Ohne gültigen Token antwortet die API mit `401` (`missing-token`, `invalid-token`, `token-expired`) oder `403` (`token-blocked`, `pk-not-allowed`, `origin-missing`, `origin-not-allowed`, `app-not-allowed`, `ip-not-allowed`) als `application/problem+json`, siehe [Fehler und Limits](/de/api/errors/). In der Übergangsphase kann der Token noch optional sein: Anfragen ohne Token gelingen und tragen den Header `X-Mapx-Token-Warning`.

## Rate Limits

Die Limits gelten pro Token und Zone (Anfragen ohne gültigen Token zählen pro IP-Adresse). Jede Antwort enthält einen `RateLimit-Policy`-Header, zum Beispiel `"search";q=10;w=1;burst=30` (10 Anfragen pro Sekunde plus Burst von 30). Bei Überschreitung folgt `429` mit `Retry-After`. Zusätzlich sind je Token standardmäßig 6000 Anfragen pro Minute erlaubt; abgelehnte Anfragen (`rate-limited`) werden gezählt.

| Zone | Anfragen | Verwendet von |
|---|---|---|
| `tiles` | 100/s, Burst 200 | Tiles, TileJSON, Glyphs, Sprites |
| `search` | 10/s, Burst 30 | Geocoding, Search Box, Autofill, Tilequery, Grundstück, Abfahrten, Wetter, Verkehr |
| `route` | 5/s, Burst 20 | Directions, Transit-Verbindungen |
| `elevation` | 10/s, Burst 20 | Höhe und Profil |
| `static` | 5/s, Burst 60 | Static Tiles, Verkehrs-Tiles |
| `heavy` | 30/min, Burst 5 | Static Images, Isochrone, Matrix, Matching, Optimization, Batch-Geocoding |

Für Massenabfragen die [PMTiles](/de/tilesets/)-Datei einmal herunterladen statt Millionen Tiles anzufordern. Details und Hinweise zum Wiederholen: [Fehler und Limits](/de/api/errors/).

## CORS und Caching

CORS erlaubt jeden Origin und den Header `Authorization`. Der Header `Authorization: Bearer` funktioniert daher auch im Browser (der Preflight wird ohne Token beantwortet), ebenso `access_token` in der URL. Die APIs lassen sich direkt aus dem Browser aufrufen. Antworten ohne Token in der URL dürfen von gemeinsamen Caches gespeichert werden; Antworten mit dem Token in der URL (Styles mit `access_token`) sind `private`.

## Attribution

Die Datenquelle überall anzeigen, wo die Karte oder das Ergebnis sichtbar ist. MapLibre fügt den Text automatisch hinzu, wenn ein Style oder TileJSON geladen wird. Siehe [Attribution](/de/reference/attribution/).

## Nächste Schritte

- [Erste Karte in 5 Minuten](/de/start/first-map/)
- [Fehler und Limits](/de/api/errors/)
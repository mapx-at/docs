# API-Token, Zugang und Rate Limits

Source: https://docs.mapx.at/de/start/access-and-limits/

> Kostenlosen mapx-API-Token holen, öffentliche und geheime Tokens unterscheiden, Übergabe des Tokens, Rate Limits pro Zone, CORS und Caching.

## Token holen

Alle Endpunkte unter `https://api.mapx.at/v1/`, `https://routing.mapx.at/`, `https://search.mapx.at/` und alle Tiles unter `https://tiles.mapx.at/` brauchen einen Token. Er ist kostenlos, es gibt keine nutzungsabhängige Verrechnung.

1. [account.mapx.at](https://account.mapx.at/) öffnen und mit der E-Mail-Adresse registrieren (ein 8-stelliger Code kommt per Mail) oder mit Google bzw. Microsoft anmelden. Der Kontaktaufnahme zu Dienst und Konto (Sicherheit, Änderungen, Abschaltung) ist zuzustimmen. Werbung und Newsletter sind eine eigene, freiwillige Checkbox.
2. Unter **Tokens** mit **Neuer Token** einen benannten Token anlegen (bis zu 100 je Konto). Der Token wird nur einmal angezeigt, dann kopieren.
3. Den Token bei jeder Anfrage mitschicken (siehe unten).

### Tokens und Konto verwalten

- **Bearbeiten:** Name, erlaubte Origins bzw. Android-App, IP-Bereiche und Ablaufdatum lassen sich nachträglich ändern; Änderungen wirken sofort. Platzhalter brauchen eine eigene Domain dahinter (`*.example.com`, nicht `*.at`), ein IP-Bereich `/0` ist nicht erlaubt (Feld dann leer lassen).
- **Ablaufdatum:** wird in UTC angegeben. Ein gesetztes Ablaufdatum lässt sich nur verkürzen, nicht verlängern.
- **Rotieren:** erzeugt einen neuen Token mit denselben Einstellungen; der alte funktioniert noch 7 Tage, damit du ihn in Ruhe austauschen kannst.
- **Sperren:** Der Token wird sofort abgelehnt (`token-blocked`) und lässt sich wieder entsperren. **Löschen** wirkt sofort und endgültig.
- **Nutzung:** Die Seiten **Übersicht** und **Nutzung** zeigen Anfragen je Token, Produkt und Tag, abgelehnte mit Grund (z. B. `origin-not-allowed`). „Zuletzt genutzt“ wird etwa einmal pro Minute aktualisiert.
- **Konto:** Unter **Konto** kannst du deine Adresse ändern (kurz nach einer Anmeldung, ein Code geht an die neue Adresse), den Newsletter an- oder abschalten, alle Daten als JSON exportieren und das Konto löschen. Mit dem Konto werden alle Tokens gelöscht; Anwendungen, die sie nutzen, funktionieren dann sofort nicht mehr.

Die OpenAPI-Beschreibung `https://api.mapx.at/v1/openapi.json` und CORS-Preflights (`OPTIONS`) funktionieren ohne Token.

## Token-Arten

| Art | Präfix | Einsatz | Einschränkung |
|---|---|---|---|
| Öffentlich | `mx_pk_…` | Webseiten und Apps. Der Token ist im Quelltext bzw. in der APK sichtbar und wirkt deshalb nur dort, wo es erlaubt ist. | Nur lesende Produkte, nur `GET`/`HEAD`. Nicht für `POST /v1/optimization`, `POST /v1/geocode/batch` und `/v1/corridor-graph` (sonst `403 pk-not-allowed`). Gebunden an **erlaubte Origins** (etwa `https://example.com`, `*.example.com`) oder an eine **Android-App** (Paketname plus SHA-256 des Signaturzertifikats). |
| Geheim | `mx_sk_…` | Server, Skripte und CI. Nie in eine Webseite oder App einbauen. | Keine Origin-Bindung; optional an IP-Bereiche (CIDR) gebunden. |
| KI-Assistent (MCP) | `mx_mcp_…` | KI-Assistenten und Agenten, die den [mapx-MCP-Server](https://docs.mapx.at/de/guides/mcp.md) nutzen. | Entsteht von Hand oder durch die Anmeldung über das Konto (erscheint dann als „verbunden über <Host>“). Gilt nur am `https://mcp.mapx.at/mcp`, keine Origin- oder IP-Bindung, optional ein Ablaufdatum. Direkte API-Aufrufe damit ergeben `403` `mcp-token-only`. |

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

**Mit dem SDK.** `mapx.map({ accessToken: "DEIN_TOKEN", … })` oder einmal `mapx.setAccessToken("DEIN_TOKEN")`; Helfer, Styles, Tiles und `staticImageUrl()` tragen den Token dann mit. Siehe [SDK](https://docs.mapx.at/de/guides/sdk/index.md).

**PMTiles-Dateien.** Token an die Datei-URL hängen: `pmtiles://https://tiles.mapx.at/<datei>.pmtiles?access_token=DEIN_TOKEN`.

**Android.** `Authorization: Bearer <öffentlicher Token>` plus `X-Mapx-Android-Package` (Paketname) und `X-Mapx-Android-Cert` (SHA-256 des Signaturzertifikats, zur Laufzeit per `PackageManager` gelesen).

Ohne gültigen Token antwortet die API mit `401` (`missing-token`, `invalid-token`, `token-expired`) oder `403` (`token-blocked`, `pk-not-allowed`, `origin-missing`, `origin-not-allowed`, `app-not-allowed`, `ip-not-allowed`) als `application/problem+json`, siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

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

**Über MCP.** Jeder Werkzeugaufruf des [MCP-Servers](https://docs.mapx.at/de/guides/mcp.md) zählt wie ein API-Aufruf (manche Werkzeuge lösen mehrere aus) und erscheint in der Nutzungsansicht als MCP. Es gelten das Minutenlimit des Tokens und die Zonenlimits oben, je Token statt je IP-Adresse. Der MCP-Endpunkt selbst nimmt 10 Anfragen pro Sekunde je IP-Adresse an (Burst 40) und Anfragekörper bis 256 KB.

Für Massenabfragen die [PMTiles](https://docs.mapx.at/de/tilesets/index.md)-Datei einmal herunterladen statt Millionen Tiles anzufordern. Details und Hinweise zum Wiederholen: [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

## CORS und Caching

CORS erlaubt jeden Origin und den Header `Authorization`. Der Header `Authorization: Bearer` funktioniert daher auch im Browser (der Preflight wird ohne Token beantwortet), ebenso `access_token` in der URL. Die APIs lassen sich direkt aus dem Browser aufrufen. Antworten ohne Token in der URL dürfen von gemeinsamen Caches gespeichert werden; Antworten mit dem Token in der URL (Styles mit `access_token`) sind `private`.

## Attribution

Die Datenquelle überall anzeigen, wo die Karte oder das Ergebnis sichtbar ist. MapLibre fügt den Text automatisch hinzu, wenn ein Style oder TileJSON geladen wird. Siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

## Ohne Gewähr

Alle Daten und Ergebnisse von mapx sind ohne Gewähr, die Nutzung erfolgt auf eigenes Risiko. Prüfe Ergebnisse selbst, besonders bei Adressen, Routen, Abfahrten, Wetter und Kataster. Es gelten die [Nutzungsbedingungen](https://mapx.at/nutzungsbedingungen/).

## Nächste Schritte

- [Erste Karte in 5 Minuten](https://docs.mapx.at/de/start/first-map.md)
- [Fehler und Limits](https://docs.mapx.at/de/api/errors.md)

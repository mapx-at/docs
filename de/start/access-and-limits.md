# API-Token, Zugang und Rate Limits

Source: https://docs.mapx.at/de/start/access-and-limits/

> Kostenlosen mapx-API-Token holen, öffentliche und geheime Tokens unterscheiden, Übergabe des Tokens, Rate Limits pro Zone, CORS und Caching.

## Token holen

Alle Endpunkte unter `https://api.mapx.at/v1/` und alle Kacheln unter `https://tiles.mapx.at/` brauchen einen Token. Er ist kostenlos.

1. [account.mapx.at](https://account.mapx.at/) öffnen und mit der E-Mail-Adresse anmelden (ein 8-stelliger Code kommt per Mail) oder mit Google bzw. Microsoft anmelden. Gibt es zur Adresse noch kein Konto, legst du es nach dem Code mit „Konto anlegen“ an; dazu hakst du die Nutzungsbedingungen, die Datenschutzerklärung und die Kontaktaufnahme zu Dienst und Konto (Sicherheit, Änderungen, Abschaltung) ab. Den Newsletter kannst du dort oder später im Konto freiwillig wählen. Code-Mails sind je Adresse und je Anschluss begrenzt; kommt die Meldung „Zu viele Versuche“, etwas später erneut versuchen.
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
| KI-Assistent (MCP) | `mx_mcp_…` | KI-Assistenten und Agenten, die den [mapx-MCP-Server](https://docs.mapx.at/de/guides/mcp.md) nutzen. | Entsteht von Hand oder durch die Anmeldung über das Konto (erscheint dann als „verbunden über &lt;Host&gt;“). Gilt nur am `https://mcp.mapx.at/mcp`, keine Origin- oder IP-Bindung, optional ein Ablaufdatum. Direkte API-Aufrufe damit ergeben `403` `mcp-token-only`. |

Ein öffentlicher Token wird gegen den Header `Origin` geprüft (bei Kachel- und Bildanfragen ohne Origin gegen den Host des `Referer`). Anfragen von anderen Seiten bekommen `403` mit dem Grund `origin-not-allowed`. Ein Origin besteht aus Schema, Host und Port, beim Entwickeln zum Beispiel `http://localhost:3000`; `localhost` nur während der Entwicklung eintragen. Eine Seite, die als Datei geöffnet wird (`file://`), hat keinen Origin: lokal einen Webserver verwenden (`npx serve`).

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
// die mapx-Stile laden ihre Kacheln aus PMTiles-Dateien: Protokoll einmal registrieren
maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile);
const map = new maplibregl.Map({
  attributionControl: false,
  container: "map",
  style: "https://api.mapx.at/v1/styles/streets",
  center: [16.3725, 48.2085], zoom: 13, // Wien, lon,lat
  transformRequest: (url) => ({
    url: /^https:\/\/([\w-]+\.)?mapx\.at\//.test(url) && !url.includes("access_token=") ? url + (url.includes("?") ? "&" : "?") + "access_token=" + TOKEN : url,
  }),
});
map.addControl(new maplibregl.AttributionControl({ compact: true }));
map.once("load", () => map.getContainer().querySelector(".maplibregl-compact-show")?.classList.remove("maplibregl-compact-show")); // Quellen eingeklappt starten (MapLibre lässt sie bis zur ersten Kartenbewegung offen)
```

**Voraussetzung PMTiles.** Die mapx-Stile laden ihre Kacheln aus PMTiles-Dateien. Die Seite muss deshalb `pmtiles.js` einbinden (`<script src="https://unpkg.com/pmtiles@4.5.0/dist/pmtiles.js"></script>`) und einmal `maplibregl.addProtocol("pmtiles", new pmtiles.Protocol().tile)` aufrufen, sonst bleibt die Karte leer (Konsole: `URL scheme "pmtiles" is not supported`). Eigene Stile, die nur TileJSON-Quellen (`https://tiles.mapx.at/v1/tiles/{tileset}.json`) verwenden, brauchen das nicht.

Wird ein Stil mit `?access_token=…` angefordert, schreibt die API den Token in alle `mapx.at`-URLs des Stils (Kacheln, Glyphs, Sprites, PMTiles-Dateien). Deshalb hängt `transformRequest` oben den Token nur an, wenn `access_token` in der URL noch fehlt. Clients ohne `transformRequest` kommen mit `style: "https://api.mapx.at/v1/styles/streets?access_token=DEIN_TOKEN"` aus (das PMTiles-Protokoll brauchen sie weiterhin). Diese Antwort ist `private` und darf nicht geteilt werden.

**Android.** `Authorization: Bearer <öffentlicher Token>` plus `X-Mapx-Android-Package` (Paketname) und `X-Mapx-Android-Cert` (SHA-256 des Signaturzertifikats, zur Laufzeit per `PackageManager` gelesen).

Ohne gültigen Token antwortet die API mit `401` (`missing-token`, `invalid-token`, `token-expired`) oder `403` (`token-blocked`, `pk-not-allowed`, `origin-missing`, `origin-not-allowed`, `app-not-allowed`, `ip-not-allowed`) als `application/problem+json`, siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

## Rate Limits

Die Zonen unten gelten je IP-Adresse (über MCP je Token). Zusätzlich gilt je Token ein Minutenlimit von standardmäßig 6000 Anfragen; abgelehnte Anfragen werden mitgezählt und erscheinen in der Nutzungsansicht beim Token der Anfrage, als `rate-limited` (Minutenlimit) bzw. `rate-limited:<Zone>` (Zonenlimit, z. B. `rate-limited:search`). Jede API-Antwort enthält einen `RateLimit-Policy`-Header, zum Beispiel `"search";q=10;w=1;burst=30` (10 Anfragen pro Sekunde plus Burst von 30); das Gateway setzt ihn aus derselben Tabelle wie seine Limits, er nennt also immer die Zone, die tatsächlich gilt. Die PMTiles-Abrufe der Stile (`https://tiles.mapx.at/…pmtiles`) tragen ihn nicht, sie zählen in die Zone `tiles`. Bei Überschreitung folgt `429` mit `Retry-After`. Vorgesehen ist außerdem ein Tages- und ein Monatslimit je Konto (alle Tokens zusammen, UTC, Überschreitung: `429` `quota-exceeded` mit `Retry-After` bis zum nächsten Tag bzw. Monat); derzeit sind beide unbegrenzt. Den Stand siehst du im Konto unter Nutzung.

| Zone | Anfragen | Verwendet von |
|---|---|---|
| `tiles` | 100/s, Burst 200 | Kacheln, TileJSON, Glyphs und Sprites (Burst 100), PMTiles-Abrufe der Stile (auf `tiles.mapx.at` Burst 500) |
| `search` | 10/s, Burst 30 | Stile und die Listen `/v1/styles`, `/v1/tiles`, Geocoding, Search Box, Autofill, Adressprüfung, Tilequery, Abfahrten, Haltestellen, Wetterwerte, Verkehrsmeldungen |
| `route` | 5/s, Burst 20 | Directions, ÖV-Verbindungen, Tempolimits (Korridorgraph: Burst 10) |
| `elevation` | 10/s, Burst 20 | Höhe und Profil |
| `static` | 5/s, Burst 60 | Static Tiles, Verkehrs-Kacheln |
| `weather` | 20/s, Burst 200 | Wetter-Kacheln (eine Radar-Animation lädt 70 bis 150 Kacheln auf einmal) |
| `heavy` | 30/min, Burst 5 | Statische Kartenbilder, Isochrone, Matrix, Matching, Optimized Trips, Optimization (Burst 3) |
| `batch` | 2/min, Burst 2 | Batch-Geocoding (bis 1000 Adressen je Anfrage, eine Anfrage gleichzeitig) |

Teure Gruppen (gerenderte Kacheln und Kartenbilder, Routing, Suche, Batch-Geocoding) haben zusätzlich ein Gesamtlimit für alle Clients zusammen, damit ein einzelner Client den Server nicht auslasten kann. Es antwortet ebenfalls mit `429` und `Retry-After`.

**Über MCP.** Jeder Werkzeugaufruf des [MCP-Servers](https://docs.mapx.at/de/guides/mcp.md) zählt wie ein API-Aufruf (manche Werkzeuge lösen mehrere aus) und erscheint in der Nutzungsansicht als MCP. Es gelten das Minutenlimit des Tokens und die Zonenlimits oben, je Token statt je IP-Adresse. Der MCP-Endpunkt selbst nimmt 10 Anfragen pro Sekunde je IP-Adresse an (Burst 40) und Anfragekörper bis 256 KB.

Für viele Kacheln: Antworten im Client cachen (Kacheln ändern sich nur mit neuem Stand) und bei `429` nach `Retry-After` wiederholen. Details und Hinweise zum Wiederholen: [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

## CORS und Caching

CORS erlaubt jeden Origin und den Header `Authorization`. Der Header `Authorization: Bearer` funktioniert daher auch im Browser (der Preflight wird ohne Token beantwortet), ebenso `access_token` in der URL. Die APIs lassen sich direkt aus dem Browser aufrufen. Trägt eine Anfrage beides, zählt der `Bearer`-Token; ein anderes `Authorization`-Schema (etwa `Basic` eines Proxys) verdeckt `access_token` nicht.

Alle Antworten auf Anfragen mit Token sind `Cache-Control: private`: Der Browser darf sie zwischenspeichern, gemeinsame Caches (Proxys, CDNs) nicht, damit niemand ohne Token deine Antwort bekommt. Jede `GET`-Antwort trägt ein `ETag`; zurückgeschickt als `If-None-Match` gibt es `304 Not Modified` ohne Inhalt (Kacheln, Stile und TileJSON nach Dateistand, alle anderen Antworten nach Inhalt, etwa minütlich abgefragte Verkehrsmeldungen).

## Quellenangabe

Die Datenquelle überall anzeigen, wo die Karte oder das Ergebnis sichtbar ist. MapLibre fügt den Text automatisch hinzu, wenn ein Stil oder TileJSON geladen wird. Siehe [Quellenangabe](https://docs.mapx.at/de/reference/attribution.md).

## Ohne Gewähr

Alle Daten und Ergebnisse von mapx sind ohne Gewähr, die Nutzung erfolgt auf eigenes Risiko. Prüfe Ergebnisse selbst, besonders bei Adressen, Routen, Abfahrten, Wetter. Es gelten die [Nutzungsbedingungen](https://mapx.at/nutzungsbedingungen/).

## Nächste Schritte

- [Erste Karte in 5 Minuten](https://docs.mapx.at/de/start/first-map.md)
- [Fehler und Limits](https://docs.mapx.at/de/api/errors.md)

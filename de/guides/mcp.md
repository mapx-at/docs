# MCP-Server für Österreich-Geodaten

Source: https://docs.mapx.at/de/guides/mcp/

> claude.ai, ChatGPT, Claude Code oder Cursor mit mapx verbinden: 21 lesende Werkzeuge für Suche, Routing, ÖV, Höhen, Verkehr und Wetter in Österreich.

Der mapx-MCP-Server gibt einem KI-Assistenten 21 Werkzeuge für Österreich-Geodaten: Adresssuche, Routen, öffentlicher Verkehr, Höhen, Verkehr und Wetter. Er nutzt das [Model Context Protocol](https://modelcontextprotocol.io/) (MCP), jeder MCP-Client kann sich verbinden. Alle Werkzeuge lesen nur, die Daten gelten nur für Österreich und stammen aus amtlichen offenen Daten, jede Antwort nennt ihre Quelle.

| Eigenschaft | Wert |
|---|---|
| Endpunkt | `https://mcp.mapx.at/mcp` |
| Transport | Streamable HTTP, nur `POST`, zustandslos |
| Protokollfassungen | `2026-07-28` und ältere ab `2024-11-05` |
| Anmeldung | Entweder über dein mapx-Konto (OAuth 2.1 mit PKCE, nur die Adresse eintragen) oder mit einem Token vom Typ „KI-Assistent (MCP)“, Präfix `mx_mcp_`, im Header `Authorization: Bearer …` |

## Einrichten

Es gibt zwei Wege. Die Anmeldung über dein Konto ist der einfachere: Du trägst nur die Adresse ein, ein Token wird nicht kopiert. Der Weg mit Token funktioniert mit jedem MCP-Client.

### Mit Anmeldung über dein Konto

1. Trage `https://mcp.mapx.at/mcp` als MCP-Server in deiner Anwendung ein (siehe unten).
2. Der Assistent leitet dich zu account.mapx.at. Melde dich mit deiner E-Mail-Adresse an; ohne Konto legst du nach dem Code eines an.
3. Auf der Zustimmungsseite siehst du die Anwendung, das Weiterleitungsziel, dein Konto und was der Zugriff erlaubt: nur lesende Werkzeuge, Aufrufe zählen zu deiner Nutzung, kein Zugriff auf dein Konto oder deine anderen Tokens. Bestätige die Verbindung.
4. Stelle eine Frage, die österreichische Daten braucht, zum Beispiel „Wie komme ich mit dem Rad vom Bahnhof Melk zum Stift Melk, und wie lange dauert das?“.

Die Verbindung erscheint danach unter [account.mapx.at](https://account.mapx.at/) → **Tokens** als „verbunden über &lt;Host&gt;“. Je Anwendung sind bis zu fünf Verbindungen gleichzeitig möglich, zum Beispiel Browser, Desktop und Handy. Die Verbindung läuft nicht von selbst ab.

Diese Anwendungen sind für die Anmeldung über das Konto freigegeben:

- **claude.ai** (Web, Desktop, Mobil): in den Einstellungen unter **Connectors** einen eigenen Connector hinzufügen und die Adresse eintragen. Die genaue Menüführung hängt vom Abo ab und ändert sich; sie steht in der [Hilfe von Anthropic](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp). In Team- und Enterprise-Abos legt zuerst ein Owner den Connector an.
- **Claude Code:** `claude mcp add --transport http mapx https://mcp.mapx.at/mcp`, dann in Claude Code `/mcp` eingeben und die Anmeldung für mapx starten.
- **ChatGPT:** Connectoren bzw. Apps mit eigener MCP-Adresse, je nach Abo und Einstellung verfügbar (teils nur im Entwicklermodus). Schritte und Verfügbarkeit stehen in der [Hilfe von OpenAI](https://help.openai.com/en/articles/12584461-developer-mode-and-mcp-apps-in-chatgpt).

Die Anmeldung über das Konto ist auf bekannte Anwendungen beschränkt. Für alle anderen Clients nimm den Weg mit Token; weitere Anwendungen schaltet der Support auf Anfrage frei (support@mapx.at).

### Mit Token

1. Lege unter [account.mapx.at/tokens/new?kind=mcp](https://account.mapx.at/tokens/new?kind=mcp) einen Token an. Wähle den Typ **KI-Assistent (MCP)**, vergib einen Namen und optional ein Ablaufdatum. Der Token wird einmal angezeigt; kopiere ihn dann.
2. Trage den Server in deinen Client ein (siehe unten) und setze den Token dort ein, wo die Beispiele `YOUR_MCP_TOKEN` zeigen.
3. Stelle eine Frage, die österreichische Daten braucht.

Der Token gilt nur am MCP-Server. Er hat keine Domain- oder IP-Bindung, kann ein Ablaufdatum haben und lässt sich in deinem Konto jederzeit sperren oder rotieren.

#### Clients für den Weg mit Token

Diese Clients nehmen einen festen Header, der Token steht also direkt in der Konfiguration.

**Claude Code**

```bash
claude mcp add --transport http mapx https://mcp.mapx.at/mcp --header "Authorization: Bearer YOUR_MCP_TOKEN"
```

**Cursor**

In `mcp.json`:

```json
{
  "mcpServers": {
    "mapx": {
      "url": "https://mcp.mapx.at/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

**VS Code**

In `.vscode/mcp.json`:

```json
{
  "servers": {
    "mapx": {
      "type": "http",
      "url": "https://mcp.mapx.at/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

**Gemini CLI**

In `settings.json`:

```json
{
  "mcpServers": {
    "mapx": {
      "httpUrl": "https://mcp.mapx.at/mcp",
      "headers": { "Authorization": "Bearer YOUR_MCP_TOKEN" }
    }
  }
}
```

**Anthropic-API**

Der [MCP-Connector](https://platform.claude.com/docs/en/agents-and-tools/mcp-connector) der Messages API ist in der Beta; Beta-Header und Modellname unten entsprechen dessen Dokumentation zum Zeitpunkt des Schreibens, prüfe sie dort nach.

```bash
curl https://api.anthropic.com/v1/messages \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2026-10-21" \
  -H "anthropic-beta: mcp-client-2025-11-20" \
  -d '{
    "model": "claude-opus-5-5",
    "max_tokens": 1000,
    "messages": [{"role": "user", "content": "Wie lange braucht ein Lkw von Wien nach Linz?"}],
    "mcp_servers": [
      { "type": "url", "url": "https://mcp.mapx.at/mcp", "name": "mapx", "authorization_token": "YOUR_MCP_TOKEN" }
    ],
    "tools": [ { "type": "mcp_toolset", "mcp_server_name": "mapx" } ]
  }'
```

**OpenAI-API**

Das [Remote-MCP-Werkzeug](https://developers.openai.com/api/docs/guides/tools-connectors-mcp) der Responses API. OpenAI fragt vor dem Senden von Daten an den Server um Freigabe, solange du `require_approval` nicht setzt. Der Wert `authorization` wird nicht gespeichert, sende ihn also bei jeder Anfrage mit.

```bash
curl https://api.openai.com/v1/responses \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -d '{
    "model": "YOUR_MODEL",
    "input": "Wie lange braucht ein Lkw von Wien nach Linz?",
    "tools": [
      { "type": "mcp", "server_label": "mapx", "server_url": "https://mcp.mapx.at/mcp", "authorization": "YOUR_MCP_TOKEN" }
    ]
  }'
```

## Verbindung trennen

Unter [account.mapx.at](https://account.mapx.at/) → **Tokens** findest du jede Verbindung über das Konto als „verbunden über &lt;Host&gt;“. Mit **Verbindung trennen** beendest du sie, mit **Sperren** wird sie sofort abgelehnt und lässt sich wieder entsperren. Der Assistent bekommt danach `401` und fragt bei Bedarf neu nach der Anmeldung.

## Für Entwickler

Der MCP-Server folgt der MCP-Autorisierung (OAuth 2.1):

- Protected Resource Metadata: `https://mcp.mapx.at/.well-known/oauth-protected-resource/mcp`
- Autorisierungsserver: `https://account.mapx.at` (Metadaten unter `/.well-known/oauth-authorization-server`)
- Authorization Code mit PKCE (`S256`); öffentliche Clients identifizieren sich über Client ID Metadata Documents, eine dynamische Registrierung gibt es nicht.
- Scope `mcp`. Das Zugriffstoken läuft nicht ab, es gibt keinen Refresh-Token; der Widerruf geschieht im Konto.

Limits, Zählung und Werkzeuge sind für beide Wege gleich.

## Werkzeuge

Der Assistent wählt die Werkzeuge selbst. Koordinaten sind `{lon, lat}` in WGS84; `geocode` macht aus einem Ortsnamen Koordinaten für die anderen Werkzeuge.

| Gruppe | Werkzeug | Zweck | Wichtigste Eingaben |
|---|---|---|---|
| Suche | `geocode` | Koordinaten zu Adresse, Straße, Ort, PLZ, POI oder Straßenkilometer | `query`, `near`, `types`, `limit` |
| Suche | `reverse_geocode` | Nächste Adresse oder nächster Ort zu einer Koordinate | `lon`, `lat`, `radius_m`, `types` |
| Suche | `validate_address` | Adresse gegen das amtliche Register prüfen und normalisieren | `address` oder `street`, `housenumber`, `postcode`, `place` |
| Suche | `search_places` | Orte (POI) nach Kategorie oder Name | `category`, `query`, `near`, `near_place`, `radius_m`, `limit` |
| Suche | `list_place_categories` | Kategorie-Ids für `search_places` | keine |
| Routing | `route` | Route mit Auto, Lkw, Rad oder zu Fuß mit Distanz, Zeit und Anweisungen | `profile`, `waypoints` (2 bis 25), `exclude`, `truck` |
| Routing | `isochrone` | Gebiet, das in gegebenen Minuten oder Metern erreichbar ist | `profile`, `lon`, `lat`, `minutes`, `meters`, `depart_at` |
| Routing | `travel_matrix` | Fahrzeiten zwischen 2 bis 25 Punkten | `profile`, `points`, `sources`, `destinations` |
| Routing | `optimize_stops` | Schnellste Reihenfolge von 2 bis 25 Stopps | `profile`, `stops`, `roundtrip`, `start`, `end` |
| ÖV | `transit_route` | Verbindungen zwischen zwei Punkten aus dem Fahrplan | `from`, `to`, `depart_at`, `arrive_by`, `access`, `connections` |
| ÖV | `departures` | Nächste planmäßige Abfahrten an einer Haltestelle oder um einen Punkt | `stop_id` oder `near`, `time`, `limit` |
| ÖV | `find_stops` | Haltestellen nahe einem Punkt oder in einem Rechteck, mit `stop_id` | `near`, `radius_m`, `bbox`, `limit` |
| Höhe | `elevation` | Höhe über dem Meer (Gelände- oder Oberflächenmodell) | `points`, `model` |
| Höhe | `elevation_profile` | Anstieg, Abstieg, tiefster und höchster Punkt entlang einer Linie | `path` (2 bis 200 Punkte), `samples`, `model` |
| Verkehr | `traffic_incidents` | Unfälle, Stau, Baustellen und Sperren | `bbox`, `road`, `road_class`, `types`, `planned` |
| Verkehr | `speed_limits` | Gesetzliche Tempolimits an einem Punkt, in einem Rechteck oder entlang eines Pfads | `at`, `bbox`, `path`, `buffer_m` |
| Wetter | `weather` | Aktuelle Messwerte, stündliche Vorhersage oder Nowcast | `lon`, `lat`, `kind`, `hours` |
| Wetter | `weather_warnings` | Amtliche Wetterwarnungen | `at`, `bbox`, `type`, `min_level`, `active_now` |
| Wetter | `climate` | Langfristige Klimadaten, Solarpotenzial | `lon`, `lat`, `sections` |
| Wetter | `air_quality` | Luftgüteindex und Schadstoffe | `lon`, `lat`, `hours` |
| Karte | `static_map` | Kartenbild mit Markern und Linie | `center`, `zoom`, `pins`, `line`, `style`, `width`, `height` |

`search_places` mit Freitext und `near` sucht nur im Umkreis `radius_m` (500 bis 100000 Meter, Standard 25000), sortiert nach Entfernung und sagt ausdrücklich, wenn es auf weiter entfernte Treffer ausweichen muss. Die Kategoriesuche liefert immer die nächsten zuerst und ignoriert `radius_m`. Statt der Koordinate `near` kannst du mit `near_place` einen Ortsnamen oder eine Adresse angeben (zum Beispiel „Melk“); der Server löst ihn mit einem zusätzlichen API-Aufruf auf, nimmt den ersten Treffer und nennt im Ergebnis, um welchen Ort er gesucht hat und welche weiteren Treffer es gab. Die Ortssuche liefert Name, Art, Koordinaten und Entfernung, aber keine Öffnungszeiten, Telefonnummern oder Bewertungen.

Zeitangaben im Antworttext sind österreichische Ortszeit mit Versatz, zum Beispiel `2026-11-18T08:00+01:00`; das gilt für Wetter, Warnungen, Luftqualität und Verkehrsmeldungen. ÖV-Zeiten sind ebenfalls Ortszeit. Der strukturierte Teil des Ergebnisses (`structuredContent`) behält die Zeitstempel der API.

Routing-Profile sind `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb` und `walking`; `isochrone` kennt zusätzlich `transit`.

## Beispielfragen

- „Wo liegt die Herrengasse 13 in 1010 Wien, und wo ist die nächste Tankstelle?“
- „Wie lange braucht ein Lkw mit 40 t von Wien nach Linz, und gibt es Sperren auf dem Weg?“
- „Was erreiche ich vom Bahnhof Melk in 15 Minuten zu Fuß?“
- „Was sind die nächsten Abfahrten am Bahnhof Melk?“
- „Wie hoch liegt die Gegend um den Großglockner, und wie sieht das Profil meines Wanderwegs aus?“
- „Regnet es in Innsbruck in den nächsten drei Stunden?“
- „Plane die beste Reihenfolge für diese fünf Lieferadressen und zeig sie auf einer Karte.“

## Limits und Zählung

Jeder Werkzeugaufruf zählt wie ein API-Aufruf und erscheint in deiner Nutzungsansicht als MCP. Manche Werkzeuge lösen mehrere API-Aufrufe aus, etwa die Freitextsuche in `search_places` (bis zu 11). Die Protokollnachrichten selbst (jede Anfrage an `/mcp`) stehen getrennt als Produkt `mcp` in der Nutzung; sie verbrauchen das Minutenlimit nicht. Es gelten das Minutenlimit deines Tokens (standardmäßig 6000) und die Zonenlimits der API, über MCP je Token statt je IP-Adresse ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Der MCP-Endpunkt selbst nimmt 10 Anfragen pro Sekunde je IP-Adresse an (Burst 40), Anfragekörper bis 256 KB. Je Token laufen höchstens 4 Aufrufe gleichzeitig; weitere warten oder scheitern mit einer Wartezeit. Bei Überschreitung bekommt der Assistent eine Fehlermeldung mit Wartezeit.

## Fehler

Der Assistent erhält den Fehler als Text und kann darauf reagieren. Fehlercodes und Ursachen stehen unter [Fehler und Limits](https://docs.mapx.at/de/api/errors.md), besonders [`mcp-token-only`](https://docs.mapx.at/de/api/errors.md#mcp-token-only) und [`mcp-token-required`](https://docs.mapx.at/de/api/errors.md#mcp-token-required).

## Grenzen

- Nur Österreich; die Daten stammen aus amtlichen offenen Daten, jede Antwort nennt die Quelle.
- Alle Werkzeuge lesen nur; nichts wird geändert oder gespeichert.
- ÖV-Antworten stammen aus dem Soll-Fahrplan.
- Klima, Wetter, Warnungen und Luftqualität stammen von der GeoSphere Austria, Verkehrsmeldungen von ASFINAG und EVIS.AT.
- Lange Ergebnisse werden gekürzt.

## Datenschutz

Deine Fragen und die Ergebnisse laufen über den Anbieter deines Assistenten. mapx sieht nur die Werkzeugaufrufe, wie bei normalen API-Aufrufen.

## Fehlersuche

| Symptom | Ursache und Abhilfe |
|---|---|
| `401` | Kein oder ungültiger Token. Bei Anmeldung über das Konto startet der Assistent damit die Anmeldung; nach dem Trennen oder Sperren meldet er `401` und fragt neu. Beim Weg mit Token prüfe den Header `Authorization: Bearer` und ob der Token gelöscht wurde oder abgelaufen ist. |
| Fehlerseite „Anwendung nicht zugelassen“ | Die Anwendung steht nicht auf der Liste der freigegebenen Anwendungen. Nimm den Weg mit Token oder schreibe an support@mapx.at. |
| `403` `mcp-token-required` | Am MCP-Server wurde ein öffentlicher (`mx_pk_`) oder geheimer (`mx_sk_`) Token verwendet. Lege einen Token vom Typ „KI-Assistent (MCP)“ an. |
| `403` `mcp-token-only` | Ein `mx_mcp_`-Token wurde für einen direkten API-Aufruf verwendet. Nimm dort einen geheimen Token (`mx_sk_`). |
| `429` | Ein Limit wurde überschritten. Warte die in der Meldung genannte Zeit und rufe weniger parallel auf. |
| `403` bei einem Browser-Client | Der `Origin` ist nicht erlaubt. Browser-Clients müssen von einer erlaubten Seite kommen; nimm einen Desktop-Client oder die API. |

## Nächste Schritte

- [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)
- [Fehler und Limits](https://docs.mapx.at/de/api/errors.md)

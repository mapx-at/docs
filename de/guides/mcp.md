# MCP-Server für Österreich-Geodaten

Source: https://docs.mapx.at/de/guides/mcp/

> Claude Code, Cursor, VS Code oder Gemini CLI mit mapx verbinden: 23 lesende Werkzeuge für Suche, Routing, ÖV, Höhen, Grundstücke, Verkehr und Wetter in Österreich.

Der mapx-MCP-Server gibt einem KI-Assistenten 23 Werkzeuge für Österreich-Geodaten: Adresssuche, Routen, öffentlicher Verkehr, Höhen, Katastergrundstücke, Verkehr und Wetter. Er nutzt das [Model Context Protocol](https://modelcontextprotocol.io/) (MCP), jeder MCP-Client kann sich verbinden. Alle Werkzeuge lesen nur, die Daten gelten nur für Österreich und stammen aus amtlichen offenen Daten, jede Antwort nennt ihre Quelle.

| | |
|---|---|
| Endpunkt | `https://mcp.mapx.at/mcp` |
| Transport | Streamable HTTP, nur `POST`, zustandslos |
| Protokollfassungen | `2026-07-28` und ältere ab `2024-11-05` |
| Anmeldung | Token vom Typ „KI-Assistent (MCP)“, Präfix `mx_mcp_`, im Header `Authorization: Bearer …` |

## In drei Schritten einrichten

1. Lege unter [account.mapx.at/tokens/new?kind=mcp](https://account.mapx.at/tokens/new?kind=mcp) einen Token an. Wähle den Typ **KI-Assistent (MCP)**, vergib einen Namen und optional ein Ablaufdatum. Der Token wird einmal angezeigt; kopiere ihn dann.
2. Trage den Server in deinen Client ein (siehe unten) und setze den Token dort ein, wo die Beispiele `YOUR_MCP_TOKEN` zeigen.
3. Stelle eine Frage, die österreichische Daten braucht, zum Beispiel „Welche Apotheken liegen im Umkreis von 500 m um den Bahnhof Melk?“.

Der Token gilt nur am MCP-Server. Er hat keine Domain- oder IP-Bindung, kann ein Ablaufdatum haben und lässt sich in deinem Konto jederzeit sperren oder rotieren.

## Client einrichten

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

> claude.ai und ChatGPT (Connectoren im Browser und in der App) brauchen die Anmeldung über dein Konto (OAuth). Das folgt in einem zweiten Schritt und ist in Vorbereitung; bis dahin nutze einen der Clients oben.

## Werkzeuge

Der Assistent wählt die Werkzeuge selbst. Koordinaten sind `{lon, lat}` in WGS84; `geocode` macht aus einem Ortsnamen Koordinaten für die anderen Werkzeuge.

| Gruppe | Werkzeug | Zweck | Wichtigste Eingaben |
|---|---|---|---|
| Suche | `geocode` | Koordinaten zu Adresse, Straße, Ort, PLZ, POI, Grundstück oder Straßenkilometer | `query`, `near`, `types`, `limit` |
| Suche | `reverse_geocode` | Nächste Adresse oder nächster Ort zu einer Koordinate | `lon`, `lat`, `radius_m`, `types` |
| Suche | `validate_address` | Adresse gegen das amtliche Register prüfen und normalisieren | `address` oder `street`, `housenumber`, `postcode`, `place` |
| Suche | `search_places` | Orte (POI) nach Kategorie oder Name | `category`, `query`, `near`, `radius_m`, `limit` |
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
| Grundstück | `parcel_at_point` | Katastergrundstück an einer Koordinate | `lon`, `lat`, `radius_m` |
| Grundstück | `parcel_details` | Grundstücks-Dossier: Fläche, Nutzung, Gebäude, Neigung, Adressen | `parcel_id` |
| Verkehr | `traffic_incidents` | Unfälle, Stau, Baustellen und Sperren | `bbox`, `road`, `road_class`, `types`, `planned` |
| Verkehr | `speed_limits` | Gesetzliche Tempolimits an einem Punkt, in einem Rechteck oder entlang eines Pfads | `at`, `bbox`, `path`, `buffer_m` |
| Wetter | `weather` | Aktuelle Messwerte, stündliche Vorhersage oder Nowcast | `lon`, `lat`, `kind`, `hours` |
| Wetter | `weather_warnings` | Amtliche Wetterwarnungen | `at`, `bbox`, `type`, `min_level`, `active_now` |
| Wetter | `climate` | Langfristige Klimadaten, Solarpotenzial | `lon`, `lat`, `sections` |
| Wetter | `air_quality` | Luftgüteindex und Schadstoffe | `lon`, `lat`, `hours` |
| Karte | `static_map` | Kartenbild mit Markern und Linie | `center`, `zoom`, `pins`, `line`, `style`, `width`, `height` |

`search_places` mit Freitext und `near` sucht nur im Umkreis `radius_m` (500 bis 100000 Meter, Standard 25000), sortiert nach Entfernung und sagt ausdrücklich, wenn es auf weiter entfernte Treffer ausweichen muss. Die Kategoriesuche liefert immer die nächsten zuerst und ignoriert `radius_m`.

Zeitangaben im Antworttext sind österreichische Ortszeit mit Versatz, zum Beispiel `2026-11-18T08:00+01:00`; das gilt für Wetter, Warnungen, Luftqualität und Verkehrsmeldungen. ÖV-Zeiten sind ebenfalls Ortszeit. Der strukturierte Teil des Ergebnisses (`structuredContent`) behält die Zeitstempel der API.

Routing-Profile sind `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb` und `walking`; `isochrone` kennt zusätzlich `transit`.

## Beispielfragen

- „Wo liegt die Herrengasse 13 in Wien, und wo ist die nächste Apotheke?“
- „Wie lange braucht ein Lkw mit 40 t von Wien nach Linz, und gibt es Sperren auf dem Weg?“
- „Was erreiche ich vom Bahnhof Melk in 15 Minuten zu Fuß?“
- „Was sind die nächsten Abfahrten am Bahnhof Melk?“
- „Wie hoch liegt die Gegend um den Großglockner, und wie sieht das Profil meines Wanderwegs aus?“
- „Welches Grundstück liegt bei 16.3725, 48.2085, wie groß ist es, und ist es Bauland?“
- „Regnet es in Innsbruck in den nächsten drei Stunden?“
- „Plane die beste Reihenfolge für diese fünf Lieferadressen und zeig sie auf einer Karte.“

## Limits und Zählung

Jeder Werkzeugaufruf zählt wie ein API-Aufruf und erscheint in deiner Nutzungsansicht als MCP. Manche Werkzeuge lösen mehrere API-Aufrufe aus, etwa die Freitextsuche in `search_places` (bis zu 11). Die Protokollnachrichten selbst (jede Anfrage an `/mcp`) stehen getrennt als Produkt `mcp` in der Nutzung; sie verbrauchen das Minutenlimit nicht. Es gelten das Minutenlimit deines Tokens (standardmäßig 6000) und die Zonenlimits der API, über MCP je Token statt je IP-Adresse ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)). Der MCP-Endpunkt selbst nimmt 10 Anfragen pro Sekunde je IP-Adresse an (Burst 40), Anfragekörper bis 256 KB. Je Token laufen höchstens 4 Aufrufe gleichzeitig; weitere warten oder scheitern mit einer Wartezeit. Bei Überschreitung bekommt der Assistent eine Fehlermeldung mit Wartezeit.

## Fehler

Der Assistent erhält den Fehler als Text und kann darauf reagieren. Fehlercodes und Ursachen stehen unter [Fehler und Limits](https://docs.mapx.at/de/api/errors.md), besonders [`mcp-token-only`](https://docs.mapx.at/de/api/errors.md#mcp-token-only) und [`mcp-token-required`](https://docs.mapx.at/de/api/errors.md#mcp-token-required).

## Grenzen

- Nur Österreich; die Daten stammen aus amtlichen offenen Daten, jede Antwort nennt die Quelle.
- Alle Werkzeuge lesen nur; nichts wird geändert oder gespeichert.
- ÖV-Antworten stammen aus dem Fahrplan, ohne Echtzeitdaten.
- Klima, Wetter, Warnungen und Luftqualität stammen von der GeoSphere Austria, Verkehrsmeldungen von ASFINAG und EVIS.AT.
- Lange Ergebnisse werden gekürzt.

## Datenschutz

Deine Fragen und die Ergebnisse laufen über den Anbieter deines Assistenten. mapx sieht nur die Werkzeugaufrufe, wie bei normalen API-Aufrufen.

## Fehlersuche

| Symptom | Ursache und Abhilfe |
|---|---|
| `401` | Kein oder ungültiger Token. Prüfe den Header `Authorization: Bearer` und ob der Token gelöscht wurde oder abgelaufen ist. |
| `403` `mcp-token-required` | Am MCP-Server wurde ein öffentlicher (`mx_pk_`) oder geheimer (`mx_sk_`) Token verwendet. Lege einen Token vom Typ „KI-Assistent (MCP)“ an. |
| `403` `mcp-token-only` | Ein `mx_mcp_`-Token wurde für einen direkten API-Aufruf verwendet. Nimm dort einen geheimen Token (`mx_sk_`). |
| `429` | Ein Limit wurde überschritten. Warte die in der Meldung genannte Zeit und rufe weniger parallel auf. |
| `403` bei einem Browser-Client | Der `Origin` ist nicht erlaubt. Browser-Clients müssen von einer erlaubten Seite kommen; nimm einen Desktop-Client oder die API. |

## Weiter

- [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)
- [Fehler und Limits](https://docs.mapx.at/de/api/errors.md)

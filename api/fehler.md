# Fehler, Limits und Betrieb

Fehlerformat RFC 9457, Anfragelimits, Caching, Health-Check und OpenAPI.

## Fehlerformat

Alle Endpunkte antworten bei Fehlern mit `application/problem+json` (RFC 9457), nicht mit HTML. Felder: `type` (`about:blank`), `title` (Statustext), `status`, `detail` (Erklärung).

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlen oder sind fehlerhaft |
| `404` | Tileset oder Kachel unbekannt, falsche Endung, außerhalb des Zoombereichs |
| `429` | zu viele Anfragen, `Retry-After` beachten |
| `503` | Dienst ausgelastet oder Höhenmodell nicht geladen |

## Limits

Der Dienst begrenzt Anfragen je IP (Standard: 50 pro Sekunde, Spitze 300; Höhenabfragen zählen je 32 Punkte zusätzlich eins) und antwortet darüber mit `429` und `Retry-After`.
Bei `429` mit wachsendem Abstand erneut versuchen.

## Caching

Kacheln und TileJSON tragen `ETag` und `Cache-Control`; mit `If-None-Match` antwortet der Dienst `304`. PMTiles-Dateinamen enthalten Stichtag und Hash und ändern sich nie.

## Health und OpenAPI

- `GET https://api.mapx.at/health`: Lebenszeichen für Monitoring.
- `GET https://api.mapx.at/v1/openapi.json`: Beschreibung aller Endpunkte, statische Kopie [openapi.yaml](/openapi.yaml).
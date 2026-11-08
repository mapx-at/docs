# Zugang und Limits

Kein Key nötig. Rate Limits pro Zone, CORS, Caching und Attribution.

## Kein Key

Alle Endpunkte unter `https://api.mapx.at/v1/` und alle Tiles unter `https://tiles.mapx.at/` sind öffentlich. Es gibt keine Registrierung und kein Token. CORS erlaubt jeden Origin, die APIs lassen sich also direkt aus dem Browser aufrufen.

## Rate Limits

Die Limits gelten pro IP-Adresse und Zone. Jede Antwort enthält einen `RateLimit-Policy`-Header, zum Beispiel `"search";q=10;w=1;burst=30` (10 Anfragen pro Sekunde plus Burst von 30). Bei Überschreitung folgt `429` mit `Retry-After`.

| Zone | Anfragen | Verwendet von |
|---|---|---|
| `tiles` | 100/s, Burst 200 | Tiles, TileJSON, Glyphs, Sprites |
| `search` | 10/s, Burst 30 | Geocoding, Search Box, Autofill, Tilequery, Grundstück, Abfahrten, Wetter, Verkehr |
| `route` | 5/s, Burst 20 | Directions, Transit-Verbindungen |
| `elevation` | 10/s, Burst 20 | Höhe und Profil |
| `static` | 5/s, Burst 60 | Static Tiles, Verkehrs-Tiles |
| `heavy` | 30/min, Burst 5 | Static Images, Isochrone, Matrix, Matching, Optimization, Batch-Geocoding |

Für Massenabfragen die [PMTiles](/de/tilesets/)-Datei einmal herunterladen statt Millionen Tiles anzufordern. Details und Hinweise zum Wiederholen: [Fehler und Limits](/de/api/errors/).

## Attribution

Die Datenquelle überall anzeigen, wo die Karte oder das Ergebnis sichtbar ist. MapLibre fügt den Text automatisch hinzu, wenn ein Style oder TileJSON geladen wird. Siehe [Attribution](/de/reference/attribution/).

## Keys später

mapx kann künftig optionale Zugangs-Keys einführen. Die bestehende Nutzung ohne Key funktioniert innerhalb der obigen Limits weiter; diese Seite beschreibt dann die Änderung.

## Nächste Schritte

- [Erste Karte in 5 Minuten](/de/start/first-map/)
- [Fehler und Limits](/de/api/errors/)
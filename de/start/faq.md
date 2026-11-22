# FAQ: Token, Lizenz, Aktualität der Daten

Source: https://docs.mapx.at/de/start/faq/

> Antworten zu mapx: kostenlos mit Konto und Token, kommerzielle Nutzung unter CC BY 4.0, Aktualität der Daten, Kacheln, Daten außerhalb Österreichs, Fehler 401 und 403.

## Brauche ich einen Token oder ein Konto?

Ja, ein kostenloses Konto und einen API-Token. Unter [account.mapx.at](https://account.mapx.at/) mit der E-Mail-Adresse anmelden (beim ersten Mal entsteht dabei das Konto), Token anlegen und als `Authorization: Bearer` oder `access_token` mitschicken. CORS ist offen. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md).

## Warum bekomme ich 401 oder 403?

`401` heißt: Token fehlt, ist ungültig oder abgelaufen. `403` heißt: Er ist gesperrt, ein öffentlicher Token wird von einer nicht erlaubten Website oder App verwendet (`origin-not-allowed`), oder ein öffentlicher Token ruft einen Endpunkt auf, der einen geheimen Token braucht (`pk-not-allowed`). Der `code` im Fehlerkörper (und der Header `X-Mapx-Reason`) nennt den Grund. Siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md) und [Fehlercodes](https://docs.mapx.at/de/api/errors.md#anmeldefehler).

## Kann ich mapx mit Claude, ChatGPT oder Cursor nutzen?

Ja, über den [MCP-Server](https://docs.mapx.at/de/guides/mcp.md): 22 lesende Werkzeuge für Österreich-Daten. In claude.ai, ChatGPT und Claude Code trägst du nur `https://mcp.mapx.at/mcp` ein und meldest dich mit deinem mapx-Konto an. Für Cursor, VS Code, Gemini CLI und andere Clients legst du einen Token vom Typ „KI-Assistent (MCP)“ an.

## Ist mapx kostenlos?

Ja, mit Konto und Token. Die Rate Limits gelten je IP-Adresse und Zone, dazu je Token ein Minutenlimit, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md#rate-limits).

## Ist kommerzielle Nutzung erlaubt?

Ja. Die amtlichen Daten stehen unter [CC BY 4.0](https://docs.mapx.at/de/reference/attribution.md): die Quelle nennen. Für die Sentinel-2-Kacheln gelten die Copernicus-Bedingungen; Einzelheiten stehen unter [Quellen und Lizenzen](https://docs.mapx.at/de/reference/attribution.md) und in den [Nutzungsbedingungen](https://mapx.at/nutzungsbedingungen/).

## Wie aktuell sind die Daten?

Jeder Kachelsatz hat ein Quelldatum. Es steht auf seiner Seite unter [Kachelsätze](https://docs.mapx.at/de/tilesets/index.md) und im Feld `stand` von `GET /v1/tiles`.

## Ändert sich eine URL bei neuen Daten?

API-URLs (`/v1/tiles/streets/...`, TileJSON) bleiben gleich. Über Stil und TileJSON kommen immer die aktuellen Daten.

## Kann ich die Kacheln herunterladen?

Nein. mapx liefert Karten und Kacheln über die API: Stil, TileJSON oder `{z}/{x}/{y}`, siehe [Karten](https://docs.mapx.at/de/guides/maps.md) und [QGIS](https://docs.mapx.at/de/guides/qgis.md). Ganze Kachelsatz-Dateien zum Selbsthosten gibt es nicht.

## Gibt es Daten außerhalb Österreichs?

Nur Österreich. Orthofoto und Basiskarte reichen an der Grenze ein Stück in die Nachbarländer.

## Was ist der Unterschied zwischen DTM und DSM?

Das DTM ist der nackte Boden, das DSM enthält Bäume und Gebäude. Das DTM für Steigungen und Profile verwenden, das DSM für Sichtbarkeit und Gebäudehöhen. Siehe [Höhen](https://docs.mapx.at/de/guides/elevation.md).

## Warum kommt 404 für eine Kachel?

Der Zoom liegt außerhalb des Bereichs des Kachelsatzes, die Endung passt nicht zum Typ (`pbf` Vektor, `webp` Raster) oder der Name des Kachelsatzes ist falsch. Kacheln innerhalb des Zoombereichs, aber außerhalb Österreichs kommen als `204` ohne Inhalt.

## Warum kommt 429?

Das Rate Limit der Zone wurde überschritten. `Retry-After` Sekunden warten und mit wachsender Pause wiederholen. Bei hoher Last Kacheln im Client cachen.

## Meine Punkte landen im falschen Land

Längen- und Breitengrad sind vertauscht. mapx und GeoJSON verwenden `lon,lat`. Siehe [Koordinaten](https://docs.mapx.at/de/reference/coordinates.md).

## Meine Karte ist unscharf

Rasterkacheln haben 512 px. Im Client `tileSize: 512` setzen. Siehe [Kachelschema](https://docs.mapx.at/de/reference/tile-scheme.md).

## Wo melde ich ein Problem?

An [support@mapx.at](mailto:support@mapx.at) schreiben (technische Fragen und Fehler) oder an [info@mapx.at](mailto:info@mapx.at) (alles andere).

# Namensregeln und Einheiten

Source: https://docs.mapx.at/de/reference/conventions/

> Wie mapx Felder und Parameter benennt: Einheiten ohne und mit Suffix, Zeitzonen (UTC außer Fahrplan), Punkte als [lng, lat], englisches snake_case, Fehler und Paging.

Alle mapx-APIs folgen denselben Regeln. Die Felder je Endpunkt stehen in der [API-Referenz](https://docs.mapx.at/de/api/index.md).

## Namen

- JSON-Schlüssel, Parameternamen und Enum-Werte sind englisches `snake_case` (`road_closure`, `railway_station`, `town_hall`). Nur die Mapbox-Navigationsfelder in `steps` (`bannerInstructions`, `voiceInstructions`, `distanceAlongGeometry`) sind camelCase.
- Ids von Stil-Ebenen und Quellen sind ebenfalls englisch (`water-area`, `place-peak`, Quellen `sentinel2`, `terrain`). Sprite-Icons heißen wie das `kind` des Objekts (`fuel`, `peak`, `saddle`).
- Zeitreihen heißen `hourly`, `daily` und beim 15-Minuten-Nowcast `minutely`.

## Einheiten

| Einheit | Regel | Beispiele |
|---|---|---|
| Meter | ohne Suffix | `distance`, `elevation`, `ascent`, `radius`, `buffer`, `snow_limit` |
| Sekunden | ohne Suffix | `duration` (immer eine Zahl in Sekunden) |
| Grad | ohne Suffix | Koordinaten, `wind_direction` |
| alles andere | Einheit als Suffix | `_kmh` (km/h), `_mm`, `_hpa`, `_pct`, `_c` (°C), `_ms` (m/s), `_min`, `_wm2` |

Beispiele: `limit_kmh`, `precipitation_mm`, `pressure_msl_hpa`, `max_grade_pct`, `temperature_c`, `wind_speed_ms`, `sunshine_min`.

## Zeitzonen

Alle Zeitangaben sind RFC 3339 in **UTC mit `Z`** (Messwerte, Vorhersagen, Warnungen, Verkehrsmeldungen, Sperren, Abrufzeiten), zum Beispiel `2026-11-21T16:00:00Z`.

Ausnahme: **Fahrplanzeiten im ÖV** (`departure`, `arrival`, `time` der Abfahrten und der Parameter `depart_at`) sind österreichische Ortszeit mit Offset (`2026-11-22T08:15:00+01:00`). Texte in MCP-Ergebnissen nennen ebenfalls Ortszeit mit Offset.

## Punkte

- Ein Punkt in einer Antwort ist immer **`[lng, lat]`** (Länge zuerst), wie in GeoJSON: `location`, `grid_location`, `geometry.coordinates`.
- Ein einzelner Punkt als Query-Parameter ist **`lng` plus `lat`**; ein Punkt als ein Wert ist `lng,lat`, eine Punktliste `lng,lat;lng,lat` (zum Beispiel in `/v1/directions/{profile}/{coordinates}`).
- `bbox` ist `west,south,east,north` (min. Länge, min. Breite, max. Länge, max. Breite).

Vertauschte Werte (`lat,lng`) lehnt die API mit `400` `invalid-coordinates` ab. Mehr unter [Koordinaten](https://docs.mapx.at/de/reference/coordinates.md).

## Antworten, Fehler und Paging

- Erfolgreiche Antworten haben kein `code`; `code` steht nur in Fehlern ([Problem Details](https://docs.mapx.at/de/api/errors.md), `detail` immer englisch).
- Unbekannte Query-Parameter und unbekannte Felder im JSON-Körper beantwortet die API mit `400` `unknown-parameter` und `param`.
- Seitenweise Listen liefern `next`: die vollständige URL der nächsten Seite oder `null` auf der letzten.
- Jede JSON-Antwort hat ein Feld `attribution`: `mapx.at; <Quelle> (<Lizenz>); …` ([Quellenangabe und Lizenz](https://docs.mapx.at/de/reference/attribution.md)).

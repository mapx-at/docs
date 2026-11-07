# E-Ladestationen

Ladestationen in Österreich als Vektorkacheln (Tileset ladestationen), Suchkategorie ladestation und Grundlage für das E-Auto-Routing.

> Die Nutzungsbedingungen der E-Control erlauben es nicht, das Verzeichnis als Webservice an Dritte weiterzugeben. Tileset, Suche und E-Auto-Routing sind deshalb ausgeschaltet, bis das geklärt ist.

Standorte der öffentlichen Ladestationen aus dem Ladestellenverzeichnis der E-Control. Das Verzeichnis wird stündlich abgerufen, mapx baut daraus täglich ein eigenes Tileset.

> Datenquelle: E-Control, Ladestellenverzeichnis (nationales Ladepunkteregister).

| | |
|---|---|
| Tileset | `ladestationen`, Vektor, z7 bis z14 |
| Layer | `ladestationen` (Punkte) |
| Felder | `id`, `name`, `operator`, `city`, `points` (Anzahl Ladepunkte), `max_kw` (größte Leistung), `connectors` (kommagetrennt), `min_zoom`, `sort_rank` |
| Stil | Gruppe `chargers` im Stil, standardmäßig aus: Punkte nach Leistung gefärbt (unter 50 kW, ab 50 kW, ab 150 kW), ab z13 mit kW-Beschriftung |
| Suche | [Search Box](/api/searchbox/) und Geocoding: Kategorie `ladestation` |
| Abfrage | [Tilequery](/api/tilequery/) mit `ladestationen` als Tileset |
| Routing | [E-Auto-Routing](/api/routing/#e-auto-routing) wählt Ladestopps aus denselben Daten |

Hochzoomstufen: Stationen ab 150 kW erscheinen ab z7, ab 50 kW ab z10, die übrigen ab z12 (Feld `min_zoom`).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/tilequery/ladestationen/15.33,48.23.json?radius=2000&limit=5"`} />
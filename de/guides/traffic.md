# Verkehrs-Guide: Meldungen und Ebene

Source: https://docs.mapx.at/de/guides/traffic/

> Verkehrsmeldungen, Tempolimits auf Autobahnen und eine Verkehrsebene auf der Karte zeigen und Routen um Sperren führen, mit der mapx-Verkehrs-API.

Die Verkehrsdaten decken das Autobahn- und Schnellstraßennetz (ASFINAG, EVIS.AT) und geplante Baustellen auf anderen Straßen ab. Anfragen liegen in der Zone `search`, 10 pro Sekunde, und brauchen einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Beispiele senden ihn im Header `Authorization`, den auch Browser senden dürfen.

| Aufgabe | Endpunkt |
|---|---|
| Störungen, Baustellen und Sperren auflisten | [`/v1/traffic/incidents`](https://docs.mapx.at/de/api/traffic.md) |
| Tempolimits, die Wechselverkehrszeichen auf Autobahnen anzeigen (keine Staulage) | [`/v1/traffic/sign-limits`](https://docs.mapx.at/de/api/traffic.md) |
| Positionen der Zeichen | [`/v1/traffic/signs`](https://docs.mapx.at/de/api/traffic.md) |
| Verkehr auf einer Karte zeichnen | `/v1/traffic/tiles.json` |
| Prüfen, ob die Daten aktuell sind | `/v1/traffic/status` |
| Um Sperren herum routen | Profil `driving-traffic` (auch `driving` umfährt verbindliche Sperren) |

## Störungen auflisten

```js
const url = "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&type=road_closure,construction&limit=20";
const { features, total, next } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
for (const f of features) console.log(f.properties.road, f.properties.type, f.properties.text);
```

Filter: `bbox`, `road` (`A1`, `A 1` und `A01` sind dieselbe Straße; Rampen und Äste wie `road_section` `A13.33.4` gehören zur `A13`), `road_class` (`motorway`, `expressway`, `federal`, `state`, kommagetrennt, für A-, S-, B- und L-Straßen), `type` (`accident`, `congestion`, `construction`, `lane_restriction`, `road_closure`, `road_hazard`, `miscellaneous`), `planned=true|false` und ein Zeitfenster mit `from` und `to` (Datum oder Zeitpunkt mit `Z` oder Offset wie `2026-12-01T10:00:00+01:00`; `to` vor `from` ergibt `400`). Standardmäßig kommen nur aktive Meldungen zurück; `active=false` ergänzt beendete und künftige. Eine Meldung mit `mandatory: true` ist verbindlich; ihr Text ist unverändert anzuzeigen.

Die Ergebnisse sind nach Bedeutung sortiert: zuerst Sperren und verbindliche Meldungen, dann Autobahnen, Schnellstraßen, Bundes- und Landesstraßen, dann nach Beginn. Mit `limit` (Standard 500, höchstens 1000) blättern: `next` ist die vollständige URL der nächsten Seite (mit `cursor`); aufrufen, bis `next` den Wert `null` hat. Bei `POST` enthält `next` nur den `cursor`: denselben Body an diese URL senden. `total` ist die Zahl der Treffer. Ungültige Werte für `road`, `road_class`, `type` oder `cursor` ergeben eine `400`-Problemantwort.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/traffic/incidents?road=A1&road_class=motorway&limit=100" | jq '.total, .next'
```

Jede Meldung hat `type` und `impact` (`critical` für Pkw gesperrt, `major` verbindlich, Fahrstreifensperre oder Unfall, `minor` sonst ungeplant, `low` sonst geplant), `kind` (genaue Kategorie als englisches Enum nach DATEX II: `road_closure`, `winter_closure`, `tunnel_closure`, `ramp_closure`, `lane_closure`, `roadworks`, `maintenance_works`, `repair_works`, `resurfacing_works`, `safety_upgrade_works`, `grass_cutting`, `alternating_traffic`, `temporary_traffic_lights`, `traffic_management`, `speed_restriction`, `congestion`, `accident`, `obstruction`, `hazardous_driving_conditions`, `weather`, `safety_exercise`, `route_recommendation`, `event`, `other`; dieselben Werte wie `category` der Sperren in Directions, Matrix, Isochrone und optimized-trips; deutsche Anzeige z. B. `road_closure` = Straßensperre, `roadworks` = Bauarbeiten, `maintenance_works` = Instandhaltung), `road` (Straßennummer, `null` bei Gemeindestraßen) sowie aus EVIS `road_name`, `direction` (Richtung als Text), `regions` (Bundesländer), `creator` und `transport_modes`. `closed: true` heißt für Pkw gesperrt; ein `road_closure` mit `closed: false` gilt nur für andere Verkehrsarten (z. B. Fußgänger, Fahrräder) oder der Verkehr läuft über die Gegenfahrbahn.

## Fahrstreifen auf Autobahnen

Baustellen auf Autobahnen und Schnellstraßen enthalten meist `lanes` von der ASFINAG: ein Eintrag je Richtung und Fahrbahn mit den gesperrten Fahrstreifen (`right`, `middle`, `left`, `hard_shoulder`), der Zahl offener Fahrstreifen (`open`) von `original`, `contraflow`, `road_closed` und dem vorübergehenden `speed_limit_kmh`. `direction` ist `aligned` (in Richtung steigender Betriebskilometer) oder `opposite`. `sources` nennt die Herkunft einer Meldung (`evis`, `asfinag`); `source` benennt den Datensatz (`evis-planned+asfinag`, wenn beide passen, `asfinag-roadworks` für Baustellen, die nur die ASFINAG meldet). Der Text einer EVIS-Meldung bleibt unverändert.

```js
const url = "https://api.mapx.at/v1/traffic/incidents?road=A12&type=construction";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
for (const f of features) for (const l of f.properties.lanes ?? []) console.log(f.properties.road, l.closed, l.open, l.speed_limit_kmh);
```

## Störungen entlang einer Route

Die Route als `route` übergeben (kodierte Polyline, Präzision mit `route_precision` 5 oder 6, oder GeoJSON-LineString). Die Ergebnisse sind nach der Position auf der Route sortiert. Für lange Routen `POST` mit JSON-Body verwenden.

```bash
# Route Wien–Graz als polyline6, dann die Meldungen im 50-m-Korridor
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.4395,47.0707?geometries=polyline6" \
  | jq '{route: .routes[0].geometry, route_precision: "6", type: "road_closure,construction", buffer: 50}' > body.json
curl -s -X POST -H "Authorization: Bearer DEIN_TOKEN" -H "Content-Type: application/json" https://api.mapx.at/v1/traffic/incidents --data @body.json | jq '.total'
```

`buffer` ist die halbe Korridorbreite in Metern (1 bis 1000, Standard 25).

## Verkehr auf der Karte zeigen

```js
map.on("load", () => {
  map.addSource("mx-traffic", { type: "vector", url: "https://api.mapx.at/v1/traffic/tiles.json" });
  // Ebene "sign_limits": angezeigte Tempolimits der Wechselverkehrszeichen je Autobahnabschnitt und Richtung
  map.addLayer({ id: "mx-sign-limits", type: "line", source: "mx-traffic", "source-layer": "sign_limits",
                 paint: { "line-width": 4, "line-color": "#3a5fcd" } });
  // Ebene "incidents": Meldungen und Sperren (Strecken und Punkte)
  map.addLayer({ id: "mx-incidents-lines", type: "line", source: "mx-traffic", "source-layer": "incidents",
                 filter: ["!=", ["geometry-type"], "Point"],
                 paint: { "line-width": 5, "line-color": "#b91c1c", "line-dasharray": [2, 1] } });
  map.addLayer({ id: "mx-incidents-points", type: "circle", source: "mx-traffic", "source-layer": "incidents",
                 filter: ["==", ["geometry-type"], "Point"],
                 paint: { "circle-radius": 6, "circle-color": "#b91c1c", "circle-stroke-color": "#fff", "circle-stroke-width": 1.5 } });
});
```

Die Vektorquelle hat zwei Ebenen: `sign_limits` (angezeigte Tempolimits der Wechselverkehrszeichen mit `speed_limit`, `normal_speed` und `reduction` in km/h; das ist keine Verkehrslage) und `incidents` (Meldungen und Sperren mit `id`, `type`, `impact`, `start`, `end`). Der Token wird wie bei jeder Karte per `transformRequest` übergeben, siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Die mapx-Stile (zum Beispiel `streets`) belegen den Quellennamen `traffic` schon, deshalb heißt die Quelle hier `mx-traffic`. Die Stile `traffic-day` und `traffic-night` (Nachtkarte mit für dunklen Grund aufgehellten Farben) zeichnen beide Ebenen fertig, die Tempolimits in einer neutralen Farbe. Die Kacheln liegen in der Zone `static` (5 pro Sekunde); der Client sollte sie cachen.

## Route um Sperren

```
GET /v1/directions/driving-traffic/{coordinates}
```

Die Route meidet die aktuellen verbindlichen Sperren aus den Meldungen von ASFINAG und EVIS.AT, erschwert zeitlich bedingte Sperren und listet die Meldungen entlang der Strecke in `legs[].incidents`. Eine flächendeckende Verkehrslage (gemessene Geschwindigkeiten, Stau auf allen Straßen) fließt nicht ein. Mit `incidents=true` kommt die Liste auch bei anderen Auto-Profilen.

## Nächste Schritte

- [Traffic API](https://docs.mapx.at/de/api/traffic.md)
- [Navigation](https://docs.mapx.at/de/guides/navigation.md): Maut, Lkw, Alternativen
- [Quellenangabe](https://docs.mapx.at/de/reference/attribution.md): ASFINAG und GIP.at nennen

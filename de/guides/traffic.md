# Verkehr

Die Verkehrsdaten decken das Autobahn- und Schnellstraßennetz (ASFINAG, EVIS.AT) und geplante Baustellen auf anderen Straßen ab. Anfragen liegen in der Zone `search`, 10 pro Sekunde.

| Aufgabe | Endpunkt |
|---|---|
| Störungen, Baustellen und Sperren auflisten | [`/v1/traffic/incidents`](/de/api/traffic/) |
| Tempolimits auf Wechselverkehrszeichen | [`/v1/traffic/flow`](/de/api/traffic/) |
| Positionen der Zeichen | [`/v1/traffic/signs`](/de/api/traffic/) |
| Verkehr auf einer Karte zeichnen | `/v1/traffic/tiles.json` |
| Prüfen, ob die Daten aktuell sind | `/v1/traffic/status` |
| Um Sperren herum routen | Profil `driving-traffic` |

## Störungen auflisten

```js
const url = "https://api.mapx.at/v1/traffic/incidents?bbox=16.0,48.0,16.8,48.5&type=road_closure,construction&limit=20";
const { features, total, next_cursor } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
for (const f of features) console.log(f.properties.road, f.properties.type, f.properties.text);
```

Filter: `bbox`, `road` (`A1`, `A 1` und `A01` sind dieselbe Straße), `road_class` (`motorway`, `expressway`, `federal`, `state`, kommagetrennt, für A-, S-, B- und L-Straßen), `type` (`accident`, `congestion`, `construction`, `lane_restriction`, `road_closure`, `road_hazard`, `miscellaneous`), `planned=true|false` und ein Zeitfenster mit `from` und `to` (UTC). Standardmäßig kommen nur aktive Meldungen zurück; `active=false` ergänzt beendete und künftige. Eine Meldung mit `mandatory: true` ist verbindlich; ihr Text ist unverändert anzuzeigen.

Die Ergebnisse sind nach Bedeutung sortiert: zuerst Sperren und verbindliche Meldungen, dann Autobahnen, Schnellstraßen, Bundes- und Landesstraßen, dann nach Beginn. Mit `limit` (Standard 500, höchstens 1000) und `cursor` blättern: `next_cursor` aus der Antwort übergeben oder `next` aufrufen, bis `next` den Wert `null` hat. `total` ist die Zahl der Treffer. Ungültige Werte für `road`, `road_class`, `type` oder `cursor` ergeben eine `400`-Problemantwort.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/traffic/incidents?road=A1&road_class=motorway&limit=100" | jq '.total, .next'
```

## Fahrstreifen auf Autobahnen

Baustellen auf Autobahnen und Schnellstraßen enthalten `lanes` von der ASFINAG: ein Eintrag je Richtung und Fahrbahn mit den gesperrten Fahrstreifen (`right`, `middle`, `left`, `hard_shoulder`), der Zahl offener Fahrstreifen (`open`) von `original`, `contraflow`, `road_closed` und dem vorübergehenden `speed_limit_kmh`. `direction` ist `aligned` (in Richtung steigender Betriebskilometer) oder `opposite`. `sources` nennt die Herkunft einer Meldung (`evis`, `asfinag`); `source` benennt den Datensatz (`evis-planned+asfinag`, wenn beide passen, `asfinag-roadworks` für Baustellen, die nur die ASFINAG meldet). Der Text einer EVIS-Meldung bleibt unverändert.

```js
const url = "https://api.mapx.at/v1/traffic/incidents?road=A12&type=lane_restriction";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
for (const f of features) for (const l of f.properties.lanes) console.log(f.properties.road, l.closed, l.open, l.speed_limit_kmh);
```

## Störungen entlang einer Route

Die Route als `route` übergeben (kodierte Polyline oder GeoJSON). Die Ergebnisse sind nach der Position auf der Route sortiert. Für lange Routen `POST` mit JSON-Body verwenden.

```bash
curl -s -X POST -H "Authorization: Bearer DEIN_TOKEN" https://api.mapx.at/v1/traffic/incidents -H "Content-Type: application/json" -d '{
  "route": { "type": "LineString", "coordinates": [[16.37, 48.21], [16.3, 48.19]] },
  "type": "road_closure,construction", "buffer": 50
}' | jq '.total'
```

`buffer` ist die halbe Korridorbreite in Metern (1 bis 1000, Standard 25).

## Verkehr auf der Karte zeigen

```js
map.on("load", () => {
  map.addSource("traffic", { type: "vector", url: "https://api.mapx.at/v1/traffic/tiles.json" });
  map.addLayer({ id: "traffic", type: "line", source: "traffic", "source-layer": "traffic",
                 paint: { "line-width": 4, "line-color": "#e11d48" } });
});
```

Der Token wird wie bei jeder Karte per `transformRequest` übergeben, siehe [Zugang und Limits](/de/start/access-and-limits/). Der Style `traffic-day` enthält den Layer bereits. Die Tiles liegen in der Zone `static` (5 pro Sekunde); der Client sollte sie cachen.

## Route mit Verkehrslage

```
GET /v1/directions/driving-traffic/{coordinates}
```

Die Route meidet aktuelle Sperren und listet Störungen in `legs[].incidents`. Mit `incidents=true` kommt die Liste auch bei anderen Auto-Profilen.

## Nächste Schritte

- [Traffic API](/de/api/traffic/)
- [Navigation](/de/guides/navigation/): Maut, Lkw, Alternativen
- [Attribution](/de/reference/attribution/): ASFINAG und GIP.at nennen


# Routing-Guide: Auto, Lkw, Rad, Fuß

Source: https://docs.mapx.at/de/guides/navigation/

> Routen für Auto, Lkw, Rad und zu Fuß in Österreich planen: Maut, Isochronen, Fahrzeitmatrizen, Map Matching und Tourenplanung.

Das Routing läuft auf dem österreichischen Straßengraphen (GIP.at). Koordinaten sind `lon,lat`, verbunden mit `;`. Jede Anfrage braucht einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Beispiele senden ihn im Header `Authorization`, den auch Browser senden dürfen.

| Aufgabe | Endpunkt | Zone |
|---|---|---|
| Route zwischen 2 bis 25 Punkten | `/v1/directions/{profile}/{coordinates}` | `route` |
| Gebiet, das in N Minuten erreichbar ist | `/v1/isochrone/{profile}/{coordinate}` | `heavy` |
| Reisezeiten zwischen vielen Punkten | `/v1/matrix/{profile}/{coordinates}` | `heavy` |
| Eine GPS-Spur auf Straßen einrasten | `/v1/matching/{profile}/{coordinates}` | `heavy` |
| Beste Reihenfolge von Stopps | `/v1/optimized-trips/{profile}/{coordinates}` | `heavy` |
| Touren für mehrere Fahrzeuge | `POST /v1/optimization` | `heavy` |

## Profile

| Profil | Für |
|---|---|
| `driving` | Auto |
| `driving-traffic` | Auto unter Berücksichtigung der Sperren und Verkehrsmeldungen von ASFINAG und EVIS.AT (keine flächendeckende Verkehrslage), listet Meldungen in `legs[].incidents` |
| `driving-truck` | Lkw über 3,5 t |
| `cycling` | Rad: bevorzugt Radwege, Radstreifen und beschilderte Routen, wertet Unbefestigtes und Steigungen schlechter |
| `cycling-mtb` | Mountainbike: bevorzugt Forststraßen und Singletrails, berücksichtigt die MTB-Schwierigkeit |
| `walking` | Zu Fuß |

Öffentlicher Verkehr ist eine eigene API: [ÖV](https://docs.mapx.at/de/guides/transit.md).

## Eine Route

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=true&language=de";
const { routes } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
const r = routes[0];
console.log(r.distance, "m", r.duration, "s");
```

`overview=simplified` (Standard) liefert eine leichte Geometrie, `full` jeden Stützpunkt, `false` keine. `steps=true` ergänzt Abbiege-Schritte. `alternatives=true` ergänzt bis zu zwei weitere `routes[]`. `annotations=duration,distance,speed,maxspeed,elevation` ergänzt Werte je Segment. Mit `cycling_surface=any` (Standard) werden unbefestigte Abschnitte schlechter bewertet, bleiben aber erlaubt. Höhen in Brücken und Tunneln stammen nicht aus dem Geländemodell. `maxspeed` (Schritt und Annotation) ist `{speed, unit, legal}`: `legal: true` kennzeichnet ein gesetzliches Limit (5, 10, 20 ... 130 km/h), `legal: false` eine modellierte GIP-Geschwindigkeit wie 53 oder 84 km/h, die eine Routinggeschwindigkeit und kein Limit ist und nicht als Schild angezeigt werden darf. `{unknown: true}` heißt kein Wert (viele Autobahnen). Bestehende Clients, die nur `speed` und `unit` lesen, funktionieren weiter. Schritte für Rad und zu Fuß tragen `grade_pct`; die Routenzusammenfassung enthält Anstieg, Abstieg und Höhenbereich.

## Tempolimits entlang der Route

`GET /v1/speedlimits` liefert die gesetzlichen Tempolimits des Straßennetzes als kompaktes GeoJSON, entweder in einer Box (`bbox`, höchstens 0,08 Quadratgrad, etwa 28 mal 28 km) oder in einem Korridor um eine Route (`corridor` als polyline6 plus `buffer` in Metern, Standard 30). Jeder Straßenabschnitt ist ein `LineString` (Digitalisierrichtung) mit `limit_kmh`, `direction` (`both`, `forward`, `backward`) und `id`. Abschnitte ohne gesetzliches Limit fehlen, damit die Schilderanzeige nie eine modellierte Geschwindigkeit zeigt.

```bash
# Limits in einer Box
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23" | jq '.features | length'
# Route als polyline6, dann die Limits in einem 30-m-Korridor
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving/15.6256,48.2049;15.3294,48.2272?geometries=polyline6&overview=full" | jq -j '.routes[0].geometry' > route.txt
curl -s -G -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/speedlimits" --data-urlencode "corridor@route.txt" -d buffer=30 | jq '.features | length'
```

Die Polyline ohne abschließenden Zeilenumbruch schreiben (`jq -j`). Lange Routen können die URL-Länge einer GET-Anfrage überschreiten; in Abschnitte von etwa 500 Punkten teilen. Quelle: GIP.at, CC BY 4.0.

## Offline neu berechnen

`GET /v1/corridor-graph` liefert das Straßennetz um eine Route als eine kompakte Binärdatei (gzip-Strom, `application/vnd.mapx.corridor-graph+gzip`), damit eine App ohne Netz neu routen kann, etwa nach einer falschen Abzweigung im Funkloch. Die Route als polyline6 in `polyline` übergeben, den Radius in `buffer` (100 bis 5000 m, Standard 3000) und `profile` (`driving` als Standard, `cycling` oder `walking`). Die Datei enthält Knoten, Kanten mit den Fahrzeiten des Routers, Straßennamen, Tempolimits und Abbiegeverbote; zu erwarten sind 1 bis 3 MB je 100 km Route. Zone `heavy`. Der Endpunkt braucht einen geheimen Token (`mx_sk_…`), mit einem öffentlichen antwortet die API mit `403` `pk-not-allowed`. Quelle: GIP.at, CC BY 4.0.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" -G "https://api.mapx.at/v1/corridor-graph" --data-urlencode "polyline@route.txt" -d buffer=500 -o corridor.mxcg
```

## Lkw-Routen

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving-truck/15.6256,48.2049;15.3294,48.2272?weight=40&height=4&width=2.55&length=16.5&truck_type=semi&overview=false" | jq '.routes[0] | {distance, duration, truck}'
```

Der Router meidet Straßen, die für das Fahrzeug gesperrt sind: Gewichtsklassen- und Anhängerverbote sowie Höhen-, Breiten-, Gewichts- und Längenbeschränkungen. Die Geschwindigkeit ist auf 80 km/h auf Autobahnen, 70 auf Freilandstraßen und 50 im Ortsgebiet begrenzt. Bezirksweite Verbote aus Freitext-Verordnungen werden nicht geroutet; betroffene Straßen stehen in `steps[].notices[]` mit `type: truck_restriction`.

## Maut

Autorouten enthalten `toll_road_distance` und `toll` mit Vignetten-Produkten und Sondermautstrecken. `toll_road_distance` zählt nur die Meter auf Sondermautstrecken (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg); die Vignettenpflicht steht in `toll.vignette_required` und `toll.vignette_distance`. `toll.vignette_distance` zählt die Routenmeter auf Autobahnen und Schnellstraßen des Straßengraphen, ohne Rastplätze, Sondermautstrecken und die vignettenfreien Abschnitte der ASFINAG-Daten. Lkw-Routen melden die mautpflichtigen Meter der GO-Maut in `routes[].toll.truck_go_distance`. Maut oder Autobahnen vermeiden:

```
.../directions/driving/{coordinates}?exclude=toll
.../directions/driving/{coordinates}?exclude=motorway,toll
```

Vignettenstraßen bleiben mit `exclude=toll` erlaubt. Quelle der Maut: ASFINAG, CC BY 4.0.

`exclude=motorway` und `exclude=toll` gelten für `driving`, `driving-traffic` und `driving-truck`.

`exclude=ferry` sperrt alle Fährverbindungen und gilt für jedes Profil (`driving`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`), allein oder kombiniert, z. B. `exclude=toll,ferry`; mit `driving-traffic` kommt 400. Weil diese Variante ohne Contraction Hierarchies gerechnet wird, dauert die Antwort einige zehn Millisekunden länger. Ist ein Ziel nur per Fähre erreichbar (etwa eine Insel), liefert die Anfrage keine Route.

## Sperren

`driving`, `driving-traffic` und `driving-truck` umfahren jede aktive Vollsperre aus den Verkehrsmeldungen (EVIS/ASFINAG), also jede Meldung mit `closed: true` in [`/v1/traffic/incidents`](https://docs.mapx.at/de/api/traffic.md). Es gibt zwei Arten:

| Art | Beispiel | Routing | `binding` |
|---|---|---|---|
| Verbindliche Sperre (EVIS-Vermerk mandatory, Verkehrsregelungen) | behördlich gesperrte Autobahnauffahrt | nie befahren | `true` |
| Vollsperre ohne diesen Vermerk | Wintersperre einer Passstraße, Straßensperre wegen Bauarbeiten | nur als letzter Ausweg befahren, wenn Start oder Ziel an oder hinter der Sperre liegt | `false` |

`routes[].closures_avoided` listet die Sperren, die die Route ohne Sperren durchfahren hätte (`id`, `text`, `road`, `category`, `binding`). Führt die Route trotzdem durch eine Sperre, weil Start oder Ziel an oder hinter ihr liegt, nennt sie `routes[].closures_unavoidable`; das als Warnung anzeigen. Zeitlich bedingte Regelungen (Wochentage oder Uhrzeiten nur im Text) werden nur gemieden, von `driving-traffic`, und nicht gelistet. Rad- und Fußprofile beachten die Sperren nicht, die Meldungen betreffen den Kfz-Verkehr. Quelle: ASFINAG, CC BY 4.0; Texte unverändert anzeigen.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving/14.2,48.27;14.45,48.30?overview=false" | jq '.routes[0].closures_avoided'
```

Matrix und Isochrone rechnen in denselben Profilen mit denselben Sperren. Ein Eintrag der [Matrix](https://docs.mapx.at/de/api/directions.md#travel-time--distance-matrix) hat dieselbe Dauer und Distanz wie eine Route zwischen denselben zwei Punkten, auch wenn sie nur als letzter Ausweg durch eine Sperre führt; `closures` nennt, ob Sperren berücksichtigt wurden (`applied`), wie viele (`count`, davon verbindlich `binding_count`, `binding` true, wenn mindestens eine) und den Stand der Meldungen (`fetched`). Eine [Isochrone](https://docs.mapx.at/de/api/directions.md#isochrones) reicht über keine Vollsperre hinaus, auch nicht über eine ohne Verbindlichkeitsvermerk, weil es ohne Ziel keinen letzten Ausweg gibt; `closures` (oberste Ebene wie bei Matrix und Directions) hat dieselben Felder, `closures_avoided` listet die Sperren, an denen die größte Stufe endet. `category` einer Sperre ist dasselbe englische Enum wie `kind` der [Verkehrsmeldungen](https://docs.mapx.at/de/api/traffic.md) (`road_closure`, `winter_closure`, `roadworks`, …).

## Sprach- und Bannerhinweise

Mit `steps=true` lassen sich `voice_instructions=true` und `banner_instructions=true` (alle Straßenprofile: driving, driving-truck, cycling, cycling-mtb und walking) ergänzen. Das Format folgt Mapbox: Jeder Schritt trägt die Hinweise für das Manöver an seinem Ende.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving/15.6256,48.2049;15.3294,48.2272?steps=true&voice_instructions=true&banner_instructions=true&language=de&overview=false" | jq '.routes[0].legs[0].steps[1] | {voiceInstructions, bannerInstructions}'
```

- `voiceInstructions[]`: `distanceAlongGeometry` (Meter vor dem Ende des Schritts, ab dem angesagt wird), `announcement` und `ssmlAnnouncement`. Die Ansagedistanzen hängen vom Tempo ab: auf Autobahnen 2 km, 1 km und 300 m vorher, auf Landstraßen 1 km, 400 m und 120 m, im Ort 400 m und 150 m; die letzte Ansage kommt ohne Entfernung. Zu Fuß wird 100 m und 40 m vorher angesagt, mit dem Rad 250 m und 80 m vorher. Die Texte folgen `language` (`de` oder `en`) für jedes Profil. Der erste Schritt kündigt auch die Abfahrt an.
- `bannerInstructions[]`: `primary` und optional `secondary` mit `text`, `components[]` (`type` `icon` für die Straßennummer, `text` für Namen und Zielwegweisung), `type`, `modifier`. Mit Zielwegweisung ist das Ziel der Haupttext und die Straße die Nebenzeile.

## Isochronen, Matrix, Matching

```js
// Wohin komme ich zu Fuß in 5, 10 und 15 Minuten?
const iso = await (await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();

// Reisezeiten vom ersten Punkt zu den anderen
const m = await (await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?sources=0&annotations=duration,distance", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
```

Das Isochronen-Ergebnis ist eine `FeatureCollection` aus Polygonen, das größte zuerst. Jedes Polygon trägt `contour` sowie fertige Darstellungswerte `fill` und `fillOpacity` mit; mit `contours_meters` statt `contours_minutes` entstehen Distanz-Konturen. Als `fill`-Ebene anzeigen:

```js
map.addSource("iso", { type: "geojson", data: iso });
map.addLayer({ id: "iso", type: "fill", source: "iso", paint: { "fill-color": ["get", "fill"], "fill-opacity": ["get", "fillOpacity"] } });
```

Siehe [Isochrone](https://docs.mapx.at/de/examples/isochrone.md) und [Matrix](https://docs.mapx.at/de/examples/matrix.md). Map Matching nimmt eine GPS-Spur und liefert die eingerastete Route.

## Tourenplanung

`optimized-trips` ordnet bis zu 25 Stopps (Rundreise oder fester Start und Ende). `POST /v1/optimization` plant Touren für mehrere Fahrzeuge mit Kapazitäten und Zeitfenstern. `POST /v1/optimization` braucht einen geheimen Token (`mx_sk_…`) und gehört auf den Server; mit einem öffentlichen Token antwortet die API mit `403` `pk-not-allowed`.

```bash
curl -s -X POST -H "Authorization: Bearer DEIN_TOKEN" https://api.mapx.at/v1/optimization -H "Content-Type: application/json" -d '{
  "vehicles": [{ "id": 1, "start": [16.3725, 48.2083], "end": [16.3725, 48.2083] }],
  "services": [{ "id": 1, "location": [15.6256, 48.2049], "service": 300 },
               { "id": 2, "location": [14.2858, 48.3069], "service": 300 }]
}' | jq '.summary'
```

## Fehler behandeln

`422` heißt: Die Anfrage ist gültig, aber nicht beantwortbar, zum Beispiel keine Route zwischen den Punkten. `404` `not-found`: Start oder Ziel liegt nicht am Straßennetz (weiter als 1000 m). `400` nennt den fehlerhaften Parameter in `param`. `429` nach `Retry-After` Sekunden wiederholen. Siehe [Fehler und Limits](https://docs.mapx.at/de/api/errors.md).

## Nächste Schritte

- [Directions API](https://docs.mapx.at/de/api/directions.md), [Optimization API](https://docs.mapx.at/de/api/optimization.md)
- [Verkehr](https://docs.mapx.at/de/guides/traffic.md): Störungen entlang einer Route
- [Routen-Beispiel](https://docs.mapx.at/de/examples/route.md)

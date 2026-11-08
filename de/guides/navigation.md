# Navigation

Routen für Auto, Lkw, Rad und zu Fuß, Maut, Isochronen, Matrizen, Map Matching und Tourenplanung.

Das Routing läuft auf dem österreichischen Straßengraphen (GIP.at). Koordinaten sind `lon,lat`, verbunden mit `;`.

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
| `driving-traffic` | Auto mit aktuellem Verkehr und Sperren |
| `driving-truck` | Lkw über 3,5 t |
| `cycling` | Rad: bevorzugt Radwege, Radstreifen und beschilderte Routen, wertet Unbefestigtes und Steigungen schlechter |
| `cycling-mtb` | Mountainbike: bevorzugt Forststraßen und Singletrails, berücksichtigt die MTB-Schwierigkeit |
| `walking` | Zu Fuß |

Öffentlicher Verkehr ist eine eigene API: [Transit](/de/guides/transit/).

## Eine Route

```js
const url = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=true&language=en";
const { routes } = await (await fetch(url)).json();
const r = routes[0];
console.log(r.distance, "m", r.duration, "s");
```

`overview=simplified` (Standard) liefert eine leichte Geometrie, `full` jeden Stützpunkt, `false` keine. `steps=true` ergänzt Abbiege-Schritte. `alternatives=true` ergänzt bis zu zwei weitere `routes[]`. `annotations=duration,distance,speed,maxspeed,elevation` ergänzt Werte je Segment. `maxspeed` (Schritt und Annotation) ist `{speed, unit, legal}`: `legal: true` kennzeichnet ein gesetzliches Limit (5, 10, 20 ... 130 km/h), `legal: false` eine modellierte GIP-Geschwindigkeit wie 53 oder 84 km/h, die eine Routinggeschwindigkeit und kein Limit ist und nicht als Schild angezeigt werden darf. `{unknown: true}` heißt kein Wert (viele Autobahnen). Bestehende Clients, die nur `speed` und `unit` lesen, funktionieren weiter. Schritte für Rad und zu Fuß tragen `grade_pct`; die Routenzusammenfassung enthält Anstieg, Abstieg und Höhenbereich.

## Tempolimits entlang der Route

`GET /v1/speedlimits` liefert die gesetzlichen Tempolimits des Straßennetzes als kompaktes GeoJSON, entweder in einer Box (`bbox`, höchstens 0,08 Quadratgrad, etwa 28 mal 28 km) oder in einem Korridor um eine Route (`corridor` als polyline6 plus `buffer` in Metern, Standard 30). Jeder Straßenabschnitt ist ein `LineString` (Digitalisierrichtung) mit `limit_kmh`, `direction` (`both`, `forward`, `backward`) und `id`. Abschnitte ohne gesetzliches Limit fehlen, damit die Schilderanzeige nie eine modellierte Geschwindigkeit zeigt.

```bash
# Route als polyline6, dann die Limits in einem 30-m-Korridor
curl -s "https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23" | jq '.features | length'
curl -s "https://api.mapx.at/v1/directions/driving/15.6256,48.2049;15.3294,48.2272?geometries=polyline6&overview=full" | jq -r '.routes[0].geometry' > route.txt
curl -s -G "https://api.mapx.at/v1/speedlimits" --data-urlencode "corridor@route.txt" -d buffer=30 | jq '.features | length'
```

Lange Routen können die URL-Länge einer GET-Anfrage überschreiten; in Abschnitte von etwa 500 Punkten teilen. Quelle: GIP.at, CC BY 4.0.

## Lkw-Routen

```bash
curl -s "https://api.mapx.at/v1/directions/driving-truck/15.6256,48.2049;15.3294,48.2272?weight=40&height=4&width=2.55&length=16.5&truck_type=semi&overview=false" | jq '.routes[0] | {distance, duration, truck}'
```

Der Router meidet Straßen, die für das Fahrzeug gesperrt sind: Gewichtsklassen- und Anhängerverbote sowie Höhen-, Breiten-, Gewichts- und Längenbeschränkungen. Die Geschwindigkeit ist auf 80 km/h auf Autobahnen, 70 auf Freilandstraßen und 50 im Ortsgebiet begrenzt. Bezirksweite Verbote aus Freitext-Verordnungen werden nicht geroutet; betroffene Straßen stehen in `steps[].notices[]` mit `type: truck_restriction`.

## Maut

Autorouten enthalten `toll_road`, `toll_road_m` und `toll` mit Vignetten-Produkten und Sondermautstrecken. Lkw-Routen melden die mautpflichtigen Kilometer der GO-Maut in `routes[].toll.truck_go_km`. Maut oder Autobahnen vermeiden:

```
.../directions/driving/{coordinates}?exclude=toll
.../directions/driving/{coordinates}?exclude=motorway,toll
```

Vignettenstraßen bleiben mit `exclude=toll` erlaubt. Quelle der Maut: ASFINAG, CC BY 4.0.

`exclude=motorway` und `exclude=toll` gelten für `driving`, `driving-traffic` und `driving-truck`.

`exclude=ferry` sperrt alle Fährverbindungen und gilt für jedes Profil (`driving`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`), allein oder kombiniert, z. B. `exclude=toll,ferry`; mit `driving-traffic` kommt 400. Weil diese Variante ohne Contraction Hierarchies gerechnet wird, dauert die Antwort einige zehn Millisekunden länger. Ist ein Ziel nur per Fähre erreichbar (etwa eine Insel), liefert die Anfrage keine Route.

## Sperren

Verbindliche Totalsperren aus den Verkehrsmeldungen (EVIS/ASFINAG) werden von `driving` und `driving-traffic` umfahren. Die betroffenen Straßen sind für die Routensuche gesperrt; `routes[].closures_avoided` listet die Sperren, die die Route ohne Sperren durchfahren hätte (`id`, `text`, `road`, `category`). Lässt sich eine Sperre nicht umfahren, führt die Route hindurch und `routes[].closures_unavoidable` nennt sie. Zeitlich bedingte Sperren (Wochentage, Uhrzeiten im Text) erschweren eine Straße nur und werden nicht gelistet. Quelle: ASFINAG, CC BY 4.0; Texte unverändert anzeigen.

```bash
curl -s "https://api.mapx.at/v1/directions/driving/14.2,48.27;14.45,48.30?overview=false" | jq '.routes[0].closures_avoided'
```

## Sprach- und Bannerhinweise

Mit `steps=true` lassen sich `voice_instructions=true` und `banner_instructions=true` (alle Straßenprofile: driving, driving-truck, cycling, cycling-mtb und walking) ergänzen. Das Format folgt Mapbox: Jeder Schritt trägt die Hinweise für das Manöver an seinem Ende.

```bash
curl -s "https://api.mapx.at/v1/directions/driving/15.6256,48.2049;15.3294,48.2272?steps=true&voice_instructions=true&banner_instructions=true&language=de&overview=false" | jq '.routes[0].legs[0].steps[1] | {voiceInstructions, bannerInstructions}'
```

- `voiceInstructions[]`: `distanceAlongGeometry` (Meter vor dem Ende des Schritts, ab dem angesagt wird), `announcement` und `ssmlAnnouncement`. Die Ansagedistanzen hängen vom Tempo ab: auf Autobahnen 2 km, 1 km und 300 m vorher, auf Landstraßen 1 km, 400 m und 120 m, im Ort 400 m und 150 m; die letzte Ansage kommt ohne Entfernung. Zu Fuß wird 100 m und 40 m vorher angesagt, mit dem Rad 250 m und 80 m vorher. Die Texte folgen `language` (`de` oder `en`) für jedes Profil. Der erste Schritt kündigt auch die Abfahrt an. `language=de|en` wählt die Sprache.
- `bannerInstructions[]`: `primary` und optional `secondary` mit `text`, `components[]` (`type` `icon` für die Straßennummer, `text` für Namen und Zielwegweisung), `type`, `modifier`. Mit Zielwegweisung ist das Ziel der Haupttext und die Straße die Nebenzeile.

## Isochronen, Matrix, Matching

```js
// Wohin komme ich zu Fuß in 5, 10 und 15 Minuten?
const iso = await (await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15")).json();

// Reisezeiten vom ersten Punkt zu den anderen
const m = await (await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?sources=0&annotations=duration,distance")).json();
```

Das Isochronen-Ergebnis ist eine `FeatureCollection` aus Polygonen, das größte zuerst. Mit einem `fill`-Layer anzeigen; siehe [Isochrone](/de/examples/isochrone/) und [Matrix](/de/examples/matrix/). Map Matching nimmt eine GPS-Spur und liefert die eingerastete Route.

## Tourenplanung

`optimized-trips` ordnet bis zu 25 Stopps (Rundreise oder fester Start und Ende). `POST /v1/optimization` plant Touren für mehrere Fahrzeuge mit Kapazitäten und Zeitfenstern.

```bash
curl -s -X POST https://api.mapx.at/v1/optimization -H "Content-Type: application/json" -d '{
  "vehicles": [{ "id": 1, "start": [16.3725, 48.2083], "end": [16.3725, 48.2083] }],
  "services": [{ "id": 1, "location": [15.6256, 48.2049], "service": 300 },
               { "id": 2, "location": [14.2858, 48.3069], "service": 300 }]
}' | jq '.summary'
```

## Fehler behandeln

`422` heißt: Die Anfrage ist gültig, aber nicht beantwortbar, zum Beispiel keine Route zwischen den Punkten. `400` nennt den fehlerhaften Parameter in `param`. `429` nach `Retry-After` Sekunden wiederholen. Siehe [Fehler und Limits](/de/api/errors/).

## Nächste Schritte

- [Directions API](/de/api/directions/), [Optimization API](/de/api/optimization/)
- [Verkehr](/de/guides/traffic/): Störungen entlang einer Route
- [Routen-Beispiel](/de/examples/route/)
# Routing-API: Routen, Isochronen, Matrix

Source: https://docs.mapx.at/de/api/directions/

> Routing-API für Österreich: Routen für Auto, Lkw, Rad und Fuß, Isochronen, Fahrzeitmatrizen und Map Matching, mit Parametern und Beispielen.

Routing auf dem österreichischen Straßengraphen (GIP.at). Profile: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `route` (5/s), `heavy` (30/min) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

## Tempolimits (GIP.at, nur gesetzliche) {#speed-limits-gipat-statutory-limits-only}

`GET /v1/speedlimits`

Gesetzliche Tempolimits des Straßennetzes als kompaktes GeoJSON, entweder in einem Rechteck (`bbox`, höchstens 0,08 Quadratgrad, etwa 28 x 28 km) oder in einem Korridor um eine Route (`corridor` als polyline6, z. B. `routes[].geometry` mit `geometries=polyline6`, dazu `buffer` in Metern).

Ein LineString-Objekt je Straßenabschnitt (Link) in Digitalisierungsrichtung mit `limit_kmh`, `direction` (`both`, `forward` oder `backward` bezogen auf die Geometrie) und `id` (GIP-Link-Id); unterscheiden sich die Richtungen, gibt es ein Objekt je Richtung. Geliefert werden nur gesetzliche Tempolimits (5, 10, 20 ... 130 km/h, einschließlich der allgemeinen Limits im Ortsgebiet und auf Freilandstraßen). Abschnitte ohne Wert (viele Autobahnen) und modellierte Geschwindigkeiten wie 53 oder 84 km/h fehlen. Höchstens 60.000 Objekte. Quelle: GIP.at (CC BY 4.0).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat in Grad, höchstens 0,08 Quadratgrad; statt corridor |
| `corridor` | query | string |  | – | Route als polyline6 (höchstens 20.000 Punkte); statt bbox |
| `buffer` | query | string |  | `30` | Nur mit corridor: Meter um die Route, 1 bis 500, Standard 30 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/speedlimits?bbox=15.325,48.225,15.335,48.23", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "legal_only": true,
  "attribution": "mapx.at; GIP.at (CC BY 4.0)",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            15.3294,
            48.2272
          ],
          [
            15.3301,
            48.2275
          ]
        ]
      },
      "properties": {
        "limit_kmh": 50,
        "direction": "both",
        "id": 1543486
      }
    }
  ]
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `legal_only` | boolean | immer true: nur gesetzliche Tempolimits sind enthalten |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Korridorgraph für Neuberechnung offline {#corridor-graph-for-offline-rerouting}

`GET /v1/corridor-graph`

Kompakter binärer Teilgraph des Straßennetzes um eine Route, um eine Route ohne Netz neu zu berechnen (etwa in einem Funkloch nach falschem Abbiegen). `polyline` ist die Route als polyline6 (z. B. `routes[].geometry` mit `geometries=polyline6`), `buffer` der Radius darum in Metern (Standard 3000, höchstens 5000).

Braucht einen geheimen Token (`mx_sk_`); öffentliche Tokens erhalten `403 pk-not-allowed`, im Playground daher nicht ausführbar.

Der Antwortkörper ist ein gzip-Strom (`application/vnd.mapx.corridor-graph+gzip`, kein `Content-Encoding`) einer `MXCG`-v1-Datei: Versionskopf und Graph-Id des Servergraphen, Knoten (Mikrograd), gerichtete Kanten mit Fahrzeit und Gewicht in ms (dieselben Werte wie im Router, offline gerechnete Kosten entsprechen also denen am Server), vereinfachte Linkgeometrie, Straßenname und -nummer, Straßenklasse (FRC/FOW), Höchstgeschwindigkeit mit Kennzeichen für gesetzliche Werte, Link-Kennzeichen (Kreisverkehr, Rampe, Maut, Ortsgebiet, Autobahn) und die Abbiegeverbote zwischen den Kanten einschließlich abgeleiteter Wendeverbote. Nicht befahrbare Kanten fehlen. Typisch 1 bis 3 MB pro 100 km Route (komprimiert). Quelle: GIP.at (CC BY 4.0).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `polyline` | query | string | ja | – | Route als polyline6 (höchstens 20.000 Punkte) |
| `profile` | query | string |  | – | driving (Standard), cycling oder walking |
| `buffer` | query | string |  | `3000` | Meter um die Route, 100 bis 5000, Standard 3000 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/corridor-graph?polyline=_wp~zAofsf%5C_nDouS_pR_af@&buffer=1000" -o result.bin
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/corridor-graph?polyline=_wp~zAofsf%5C_nDouS_pR_af@&buffer=1000", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const blob = await res.blob();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/corridor-graph?polyline=_wp~zAofsf%5C_nDouS_pR_af@&buffer=1000", headers={"Authorization": "Bearer YOUR_TOKEN"})
open("result.bin", "wb").write(res.content)
```

**Beispielantwort**: `application/vnd.mapx.corridor-graph+gzip` · Korridorgraph (gzip von MXCG v1)

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Routing {#routing}

`GET /v1/directions/{profile}/{coordinates}`

Lkw (`driving-truck`, über 3,5 t): Fahrzeugmaße über `weight`, `height`, `width`, `length`, `truck_type`; Straßen werden je Fahrzeug gesperrt (Gewichtsklassen-, Sattel- und Anhängerverbote, Höhen-, Breiten-, Gewichts- und Längenbeschränkungen). Lkw-Geschwindigkeiten: höchstens 80 km/h auf Autobahnen, 70 auf Freilandstraßen, 50 im Ortsgebiet.

Bezirksweite Fahrverbote (Verordnungen im Freitext) werden nicht geroutet; betroffene Straßen erscheinen als `steps[].notices[]` (`type: truck_restriction`). Antworten enthalten zusätzlich `routes[].truck` (das verwendete Fahrzeug) und `routes[].toll.truck_go_distance` (Meter mit GO-Maut; je Schritt `truck_go_distance`). 2 bis 25 Koordinaten; die Teilstrecken werden einzeln berechnet. Pkw-Routen (`driving`) enthalten `routes[].toll` (Vignetten- und Sondermautdaten), `toll_road_distance` und `steps[].intersections[].classes`; Mautquelle ASFINAG, CC BY 4.0. Aus der GIP tragen Schritte zusätzlich `ref`, `destinations`, `milestone`, `maxspeed`, `ascent`, `descent` (Meter) und (Rad/Fuß) `grade_pct`, bei cycling und cycling-mtb auch `bike_infrastructure` (cycleway, lane, shared, none) und `surface` (paved, cobblestone, unpaved, offroad, unknown); `routes[].summary` nennt die zwei längsten Straßen (String wie `legs[].summary`, mit `steps=true`), `routes[].elevation_summary` enthält Anstieg, Abstieg, größte Steigung und Höhenbereich. Der Standard `overview=simplified` liefert eine vereinfachte Geometrie, `steps=true` ergänzt Schrittgeometrie und Kreuzungen. Sperren: `driving`, `driving-traffic` und `driving-truck` umfahren die aktiven Vollsperren aus den Verkehrsmeldungen (EVIS/ASFINAG, `closed: true` in `/v1/traffic/incidents`). Verbindliche Sperren (EVIS-Vermerk mandatory, Verkehrsregelungen) werden nie befahren; Vollsperren ohne diesen Vermerk, etwa Wintersperren von Passstraßen, nur als letzter Ausweg, wenn Start oder Ziel an oder hinter der Sperre liegt. `routes[].closures_avoided` listet die Sperren, über die die Route ohne Sperren geführt hätte (`binding` unterscheidet beide Arten), `routes[].closures_unavoidable` jene, durch die die Route trotzdem führt. Zeitlich bedingte Regelungen (Wochentage oder Uhrzeiten nur im Text) werden nur gemieden, und nur von `driving-traffic`. Rad- und Fußprofile beachten die Sperren nicht (die Meldungen betreffen den Kfz-Verkehr). `driving-traffic` verlangsamt zusätzlich Autobahnabschnitte, auf denen ASFINAG-Anzeigen ein Tempolimit unter der freien Fahrgeschwindigkeit zeigen (siehe `/v1/traffic/sign-limits`); eine flächendeckende Verkehrslage gibt es nicht. Navigation: `steps=true` mit `voice_instructions=true` und/oder `banner_instructions=true` ergänzt Sprach- und Banneranweisungen im Mapbox-Format (alle Straßenprofile, Texte in `language`). Tempolimits: `maxspeed` (Schritt und Annotation) ist `{speed, unit, legal}`; `legal: true` kennzeichnet ein gesetzliches Tempolimit (5, 10, 20 ... 130 km/h), `legal: false` eine modellierte GIP-Geschwindigkeit (z. B. 53, 84), die kein Tempolimit ist; `{unknown: true}` ohne Wert (abwärtskompatibel: `speed` und `unit` bleiben unverändert). Tempolimits entlang eines Korridors: `/v1/speedlimits`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (umfährt aktive Sperren aus den EVIS/ASFINAG-Meldungen und verlangsamt Autobahnabschnitte, auf denen Anzeigen ein gesenktes Tempolimit zeigen; keine flächendeckende Verkehrslage), driving-truck (Lkw über 3,5 t; Fahrzeugmaße über weight/height/width/length/truck_type), cycling, cycling-mtb oder walking |
| `coordinates` | path | string | ja | – | lng,lat;lng,lat;... |
| `steps` | query | string |  | – | true für Abbiegehinweise (mit Manövern und Kreuzungen) |
| `geometries` | query | string |  | – | geojson (Standard) \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (Standard, vereinfachte Routengeometrie), full (alle Stützpunkte) oder false (keine Geometrie) |
| `language` | query | string |  | – | de (Standard) oder en für Anweisungen und Ansagen; andere Werte 400 |
| `alternatives` | query | string |  | – | true (bis zu 2), false oder 0 bis 3; nur mit genau 2 Koordinaten. Alternativen entstehen über seitlich versetzte Wegpunkte und erscheinen als zusätzliche `routes[]` |
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (durch Komma getrennt; `weather_warnings` ergänzt `routes[].weather_warnings`, die amtlichen Warnungen, deren Gebiet die Route quert, ohne Geometrie): `legs[].annotation` je Geometriesegment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (nur driving; `legal` ist true bei einem gesetzlichen Tempolimit und false bei einer modellierten Geschwindigkeit wie 53 oder 84, die nicht als Tempolimit angezeigt werden darf), elevation = mittlere Segmenthöhe in m |
| `radiuses` | query | string |  | – | Suchradius je Koordinate in Metern, durch Semikolon getrennt, oder `unlimited` (20 km); jede Koordinate rastet in ihrem eigenen Radius ein, sonst 404 `no-road-nearby` |
| `approaches` | query | string |  | – | unrestricted oder curb je Koordinate (wird geprüft, vom Router noch nicht verwendet) |
| `bearings` | query | string |  | – | Grad,Toleranz je Koordinate (wird geprüft, vom Router noch nicht verwendet) |
| `incidents` | query | string |  | `true for driving-traffic` | true/false: Verkehrsmeldungen entlang der Teilstrecke als `legs[].incidents`; Standard true bei driving-traffic, sonst false; nur Pkw |
| `exclude` | query | string |  | – | motorway, toll und/oder ferry, durch Komma getrennt: `motorway` meidet Autobahnen (Kategorie A; driving, driving-traffic, driving-truck); `toll` meidet alle Mautstraßen: vignettenpflichtige ASFINAG-Autobahnen und -Schnellstraßen (Kategorien A und S) und Sondermautstrecken (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg; dieselben Profile). Vereinfachung: Auch die vignettenfreien Autobahnabschnitte (z. B. A1 Walserberg bis Salzburg Nord, A12 bei Kufstein, A14 Hörbranz bis Hohenems) werden gemieden, weil der Straßengraph (GIP) kein Vignettenmerkmal hat; mit toll meldet die Route `toll.vignette_distance` 0, und gibt es keine mautfreie Route, schlägt die Anfrage fehl, statt auf eine Mautroute auszuweichen; `ferry` sperrt alle Fährverbindungen (alle Profile außer driving-traffic, das 400 liefert). Mit `ferry` wird die Route ohne Contraction Hierarchies berechnet und dauert daher einige zehn Millisekunden länger |
| `voice_instructions` | query | string |  | – | true: `steps[].voiceInstructions` (Mapbox-Format mit announcement und ssmlAnnouncement, Sprache nach `language`); erfordert steps=true; alle Straßenprofile (driving, driving-truck, cycling, cycling-mtb, walking), mit kürzeren Ansageabständen zu Fuß (100 m, 40 m) und mit dem Rad (250 m, 80 m) |
| `banner_instructions` | query | string |  | – | true: `steps[].bannerInstructions` (Mapbox-Format, primary/secondary mit Text und Komponenten; Wegweiser und Straßennummer); erfordert steps=true; alle Straßenprofile |
| `cycling_surface` | query | string |  | – | any (Standard), nur cycling; paved (unbefestigte Abschnitte ganz sperren) ist derzeit nicht verfügbar und ergibt 400 |
| `weight` | query | string |  | `40` | Nur driving-truck: Gesamtgewicht in t (Standard 40, 0.5 bis 60); meidet Gewichtsklassenverbote und Gewichtsbeschränkungen |
| `height` | query | string |  | `4` | Nur driving-truck: Fahrzeughöhe in m (Standard 4, 1.5 bis 5); meidet Höhenbeschränkungen |
| `width` | query | string |  | `2` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
| `length` | query | string |  | – | Nur driving-truck: Fahrzeuglänge in m (2 bis 40); Längenbeschränkungen werden nur geprüft, wenn angegeben |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | Nur driving-truck: semi (Sattelzug, Standard), rigid (Lkw ohne Anhänger) oder trailer (Lkw mit Anhänger) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?steps=false&overview=simplified", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "routes": [
    {
      "weight_name": "routability",
      "weight": 1850.2,
      "distance": 62340.1,
      "duration": 2890.4,
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3725,
            48.2083
          ],
          [
            15.6256,
            48.2049
          ]
        ]
      },
      "legs": [
        {
          "distance": 62340.1,
          "duration": 2890.4,
          "weight": 1850.2,
          "summary": "A1, S33"
        }
      ],
      "summary": "A1, S33"
    }
  ],
  "waypoints": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ],
      "distance": 4.2
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ],
      "distance": 6.1
    }
  ],
  "attribution": "mapx.at; GIP.at (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Route mit Query-Parametern {#directions-with-query-parameters}

`GET /v1/directions`

Gleiche Antwort wie `/v1/directions/{profile}/{coordinates}` (bevorzugt); alle Query-Parameter der Pfadform gelten.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | query | string |  | `driving` | Standard driving |
| `coordinates` | query | string | ja | – | lng,lat;lng,lat;... |
| `steps` | query | string |  | – | true für Abbiegehinweise (mit Manövern und Kreuzungen) |
| `geometries` | query | string |  | – | geojson (Standard) \| polyline \| polyline6 |
| `overview` | query | string |  | – | simplified (Standard, vereinfachte Routengeometrie), full (alle Stützpunkte) oder false (keine Geometrie) |
| `language` | query | string |  | – | de (Standard) oder en für Anweisungen und Ansagen; andere Werte 400 |
| `alternatives` | query | string |  | – | true (bis zu 2), false oder 0 bis 3; nur mit genau 2 Koordinaten. Alternativen entstehen über seitlich versetzte Wegpunkte und erscheinen als zusätzliche `routes[]` |
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (durch Komma getrennt; `weather_warnings` ergänzt `routes[].weather_warnings`, die amtlichen Warnungen, deren Gebiet die Route quert, ohne Geometrie): `legs[].annotation` je Geometriesegment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (nur driving; `legal` ist true bei einem gesetzlichen Tempolimit und false bei einer modellierten Geschwindigkeit wie 53 oder 84, die nicht als Tempolimit angezeigt werden darf), elevation = mittlere Segmenthöhe in m |
| `radiuses` | query | string |  | – | Suchradius je Koordinate in Metern, durch Semikolon getrennt, oder `unlimited` (20 km); jede Koordinate rastet in ihrem eigenen Radius ein, sonst 404 `no-road-nearby` |
| `approaches` | query | string |  | – | unrestricted oder curb je Koordinate (wird geprüft, vom Router noch nicht verwendet) |
| `bearings` | query | string |  | – | Grad,Toleranz je Koordinate (wird geprüft, vom Router noch nicht verwendet) |
| `incidents` | query | string |  | `true for driving-traffic` | true/false: Verkehrsmeldungen entlang der Teilstrecke als `legs[].incidents`; Standard true bei driving-traffic, sonst false; nur Pkw |
| `exclude` | query | string |  | – | motorway, toll und/oder ferry, durch Komma getrennt: `motorway` meidet Autobahnen (Kategorie A; driving, driving-traffic, driving-truck); `toll` meidet alle Mautstraßen: vignettenpflichtige ASFINAG-Autobahnen und -Schnellstraßen (Kategorien A und S) und Sondermautstrecken (A9 Gleinalm/Bosruck, A10 Tauern/Katschberg, A11 Karawanken, A13 Brenner, S16 Arlberg; dieselben Profile). Vereinfachung: Auch die vignettenfreien Autobahnabschnitte (z. B. A1 Walserberg bis Salzburg Nord, A12 bei Kufstein, A14 Hörbranz bis Hohenems) werden gemieden, weil der Straßengraph (GIP) kein Vignettenmerkmal hat; mit toll meldet die Route `toll.vignette_distance` 0, und gibt es keine mautfreie Route, schlägt die Anfrage fehl, statt auf eine Mautroute auszuweichen; `ferry` sperrt alle Fährverbindungen (alle Profile außer driving-traffic, das 400 liefert). Mit `ferry` wird die Route ohne Contraction Hierarchies berechnet und dauert daher einige zehn Millisekunden länger |
| `voice_instructions` | query | string |  | – | true: `steps[].voiceInstructions` (Mapbox-Format mit announcement und ssmlAnnouncement, Sprache nach `language`); erfordert steps=true; alle Straßenprofile (driving, driving-truck, cycling, cycling-mtb, walking), mit kürzeren Ansageabständen zu Fuß (100 m, 40 m) und mit dem Rad (250 m, 80 m) |
| `banner_instructions` | query | string |  | – | true: `steps[].bannerInstructions` (Mapbox-Format, primary/secondary mit Text und Komponenten; Wegweiser und Straßennummer); erfordert steps=true; alle Straßenprofile |
| `cycling_surface` | query | string |  | – | any (Standard), nur cycling; paved (unbefestigte Abschnitte ganz sperren) ist derzeit nicht verfügbar und ergibt 400 |
| `weight` | query | string |  | `40` | Nur driving-truck: Gesamtgewicht in t (Standard 40, 0.5 bis 60); meidet Gewichtsklassenverbote und Gewichtsbeschränkungen |
| `height` | query | string |  | `4` | Nur driving-truck: Fahrzeughöhe in m (Standard 4, 1.5 bis 5); meidet Höhenbeschränkungen |
| `width` | query | string |  | `2` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
| `length` | query | string |  | – | Nur driving-truck: Fahrzeuglänge in m (2 bis 40); Längenbeschränkungen werden nur geprüft, wenn angegeben |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | Nur driving-truck: semi (Sattelzug, Standard), rigid (Lkw ohne Anhänger) oder trailer (Lkw mit Anhänger) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/directions?profile=driving&coordinates=16.3725,48.2085;15.3294,48.2272", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "routes": [
    {
      "weight_name": "routability",
      "weight": 1850.2,
      "distance": 62340.1,
      "duration": 2890.4,
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [
            16.3725,
            48.2083
          ],
          [
            15.6256,
            48.2049
          ]
        ]
      },
      "legs": [
        {
          "distance": 62340.1,
          "duration": 2890.4,
          "weight": 1850.2,
          "summary": "A1, S33"
        }
      ],
      "summary": "A1, S33"
    }
  ],
  "waypoints": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ],
      "distance": 4.2
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ],
      "distance": 6.1
    }
  ],
  "attribution": "mapx.at; GIP.at (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Isochronen {#isochrones}

`GET /v1/isochrone/{profile}/{coordinate}`

Erreichbare Gebiete, größtes zuerst; `properties.contour/color/opacity`. Zone `heavy`. Das Profil `transit` rechnet zu Fuß und mit öffentlichem Verkehr (GTFS-Fahrplan, RAPTOR): zu Fuß zu den Haltestellen, Fahrt ab `depart_at`, von jeder erreichten Haltestelle zu Fuß weiter; nur `contours_minutes` (Standard 5,10,15, höchstens 120 min), es gilt der Gültigkeitszeitraum des Fahrplans (sonst 422), `metadata` enthält `depart_at` (Ortszeit mit Offset) und `transit_stops`.

`access=foot|bike|car` (Standard `foot`) legt fest, wie die erste Haltestelle erreicht wird: `bike` = Bike & Ride (mit dem Rad zu jeder Haltestelle, +2 min zum Absperren), `car` = Park & Ride (mit dem Auto, höchstens 25 min, zu Haltestellen mit Bahn, U-Bahn oder Straßenbahn, +3 min zum Parken, und zu amtlichen P+R-Anlagen der GIP, +2 min und ein Fußweg zur Haltestelle); die schnellste Kombination gewinnt, weiter geht es immer zu Fuß. `park=official` beschränkt das Parken auf amtliche P+R/B+R-Anlagen. `metadata` enthält dann zusätzlich `access`, `park`, `access_stops_improved` und `access_sites` (die ersten `sites` Einträge: `source` site|stop, `kind`, `name`, `lng`, `lat`, `arrive_min`, `ready_min`, `stops_won`). Die Radmitnahme in Zügen steht nicht in den Fahrplandaten, daher gibt es kein `egress=bike`. Die anderen Profile lehnen `depart_at`, `access` und `park` ab (400). Sperren: driving, driving-traffic und driving-truck reichen nicht über aktive Vollsperren aus den Verkehrsmeldungen hinaus (dieselben, die Directions umfährt, siehe `/v1/traffic/incidents` mit `closed: true`), auch nicht über Sperren ohne Verbindlichkeitsvermerk wie Wintersperren von Passstraßen: Eine Isochrone hat kein Ziel hinter der Sperre, einen letzten Ausweg gibt es daher nicht; die Straße, auf der der Startpunkt einrastet, bleibt befahrbar. `closures` meldet, ob Sperren berücksichtigt wurden (`applied`, `count`, `binding`, `binding_count`, `fetched`), `closures_avoided` listet die Sperren, an denen die größte Stufe endet (`id`, `text`, `road`, `category`, `binding`), beide auf oberster Ebene wie bei Directions und Matrix.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic, driving-truck, cycling, cycling-mtb, walking oder transit (zu Fuß und mit öffentlichem Verkehr ab `depart_at`) |
| `coordinate` | path | string | ja | – | lng,lat |
| `contours_minutes` | query | string |  | `5` | Bis zu 4 Werte, durch Komma getrennt (Standard 5,10,15) |
| `contours_meters` | query | string |  | – | Distanzen statt Minuten (nicht für `transit`) |
| `depart_at` | query | string |  | `now` | Nur `transit`: Abfahrt, ISO 8601 in Ortszeit Europe/Vienna ohne Offset (`2026-11-10T08:00`), mit `Z`/Offset oder als Unix-Sekunden; Standard jetzt |
| `access` | query | string |  | – | Nur `transit`: foot (Standard) \| bike (Bike & Ride) \| car (Park & Ride) |
| `park` | query | string |  | `bike every stop` | Nur `transit` mit bike/car: any (Standard: Rad an jeder Haltestelle, Auto an Bahnhaltestellen) \| official (nur amtliche P+R/B+R-Anlagen) |
| `sites` | query | string |  | `10` | Nur `transit` mit bike/car: Zahl der Zugangseinträge in `metadata.access_sites` (Standard 10, höchstens 50) |
| `weight` | query | string |  | `40` | Nur driving-truck: Gesamtgewicht in t (Standard 40, 0.5 bis 60); meidet Gewichtsklassenverbote und Gewichtsbeschränkungen |
| `height` | query | string |  | `4` | Nur driving-truck: Fahrzeughöhe in m (Standard 4, 1.5 bis 5); meidet Höhenbeschränkungen |
| `width` | query | string |  | `2` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
| `length` | query | string |  | – | Nur driving-truck: Fahrzeuglänge in m (2 bis 40); Längenbeschränkungen werden nur geprüft, wenn angegeben |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | Nur driving-truck: semi (Sattelzug, Standard), rigid (Lkw ohne Anhänger) oder trailer (Lkw mit Anhänger) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/isochrone/walking/16.3725,48.2085?contours_minutes=5,10,15", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {
        "contour": 15,
        "metric": "time",
        "color": "#bf4040",
        "opacity": 0.33
      },
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
          [
            [
              [
                16.3,
                48.2
              ],
              [
                16.4,
                48.2
              ]
            ]
          ]
        ]
      }
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Fahrzeit- und Distanzmatrix {#travel-time--distance-matrix}

`GET /v1/matrix/{profile}/{coordinates}`

2 bis 25 Koordinaten. `sources`/`destinations` als Indexliste (`0;2`) oder `all`. Zone `heavy`. Sperren: driving, driving-traffic und driving-truck rechnen mit denselben Sperren und Gewichten wie Directions (verbindliche Sperren nie, übrige Vollsperren wie Wintersperren nur als letzter Ausweg, zeitlich bedingte Regelungen nur bei driving-traffic); jeder Eintrag passt daher zu Dauer und Distanz einer Directions-Anfrage zwischen denselben zwei Punkten.

`closures` meldet, ob Sperren berücksichtigt wurden (`applied`, `count`, `binding`, `binding_count`, `fetched`); cycling und walking beachten keine Sperren und haben kein `closures`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (umfährt aktive Sperren aus den EVIS/ASFINAG-Meldungen und verlangsamt Autobahnabschnitte, auf denen Anzeigen ein gesenktes Tempolimit zeigen; keine flächendeckende Verkehrslage), driving-truck (Lkw über 3,5 t; Fahrzeugmaße über weight/height/width/length/truck_type), cycling, cycling-mtb oder walking |
| `coordinates` | path | string | ja | – | lng,lat;lng,lat;... |
| `sources` | query | string |  | – | Indizes, durch Strichpunkt getrennt |
| `destinations` | query | string |  | – | Indizes, durch Strichpunkt getrennt |
| `annotations` | query | string |  | – | duration (Standard), distance oder beides |
| `weight` | query | string |  | `40` | Nur driving-truck: Gesamtgewicht in t (Standard 40, 0.5 bis 60); meidet Gewichtsklassenverbote und Gewichtsbeschränkungen |
| `height` | query | string |  | `4` | Nur driving-truck: Fahrzeughöhe in m (Standard 4, 1.5 bis 5); meidet Höhenbeschränkungen |
| `width` | query | string |  | `2` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
| `length` | query | string |  | – | Nur driving-truck: Fahrzeuglänge in m (2 bis 40); Längenbeschränkungen werden nur geprüft, wenn angegeben |
| `truck_type` | query | `semi` \| `rigid` \| `trailer` |  | – | Nur driving-truck: semi (Sattelzug, Standard), rigid (Lkw ohne Anhänger) oder trailer (Lkw mit Anhänger) |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/matrix/driving/16.3725,48.2085;15.3294,48.2272;14.2858,48.3069?annotations=duration,distance", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "durations": [
    [
      0,
      2890.4
    ],
    [
      2901.2,
      0
    ]
  ],
  "sources": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ]
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ]
    }
  ],
  "destinations": [
    {
      "name": "",
      "location": [
        16.3725,
        48.2083
      ]
    },
    {
      "name": "",
      "location": [
        15.6256,
        48.2049
      ]
    }
  ],
  "closures": {
    "applied": true,
    "count": 113,
    "binding": true,
    "binding_count": 8,
    "fetched": "2026-11-22T01:39:31Z"
  }
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `durations` | array | Sekunden |
| `distances` | array | Meter |
| `closures` | object | Berücksichtigte Straßensperren (nur driving, driving-traffic, driving-truck; fehlt bei cycling und walking). |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Map Matching {#map-matching}

`GET /v1/matching/{profile}/{coordinates}`

2 bis 100 GPS-Punkte, auf das Straßennetz eingepasst. Ohne Zeitstempel gilt ein einzelner Punkt weit abseits der Spur (hin und gleich zurück, beide Schenkel über 300 m und zusammen mehr als das Dreifache des direkten Abstands) als Ausreißer: sein Tracepoint ist `null`, das Matching läuft weiter.

Liegen aufeinanderfolgende Punkte mehr als 5 km (mit Zeitstempeln: 300 s) auseinander, teilt sich die Spur in mehrere `matchings`. Lässt sich kein Teil zuordnen, antwortet die API mit 422 (`No match`). Längere Spuren: `POST https://routing.mapx.at/match` mit einem geheimen Token (`mx_sk_…`; öffentliche Tokens erlauben nur GET) und dem JSON-Body `{"coordinates": [[lng, lat], ...], "timestamps": [...], "radius": 25}` (höchstens 512 KB); die Antwort hat ein eigenes Format, nicht das von Mapbox: eine GeoJSON-FeatureCollection mit einem LineString je eingepasstem Abschnitt und `tracepoints` (eingepasster Punkt und Konfidenz je Eingabepunkt, `null`, wenn nicht zugeordnet).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `profile` | path | string | ja | – | driving, driving-traffic (umfährt aktive Sperren aus den EVIS/ASFINAG-Meldungen und verlangsamt Autobahnabschnitte, auf denen Anzeigen ein gesenktes Tempolimit zeigen; keine flächendeckende Verkehrslage), driving-truck (Lkw über 3,5 t; Fahrzeugmaße über weight/height/width/length/truck_type), cycling, cycling-mtb oder walking |
| `coordinates` | path | string | ja | – | lng,lat;lng,lat;... |
| `timestamps` | query | string |  | – | Unix-Sekunden, durch Strichpunkt getrennt |
| `radiuses` | query | string |  | – | Ein Wert je Punkt, 5 bis 500 m, durch Semikolon getrennt (das Matching sucht Kandidaten im größten Wert); sonst 400 |
| `geometries` | query | string |  | – | geojson \| polyline \| polyline6 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/matching/driving/16.3725,48.2085;16.3680,48.2100;16.3600,48.2110", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "matchings": [],
  "tracepoints": []
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

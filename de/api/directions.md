# Directions-API: Routen für Auto, Lkw, Rad und Fuß

Source: https://docs.mapx.at/de/api/directions/

> Directions-API für Österreich: Routen für Auto, Lkw, Rad und Fuß auf dem GIP.at-Straßengraphen und der Korridorgraph für Offline-Neuberechnung, mit Beispielen.

Routing auf dem österreichischen Straßengraphen (GIP.at). Profile: `driving`, `driving-traffic`, `driving-truck`, `cycling`, `cycling-mtb`, `walking`. Isochronen, Matrix, Map Matching und Tempolimits haben eigene Seiten: [Isochrone](https://docs.mapx.at/de/api/isochrone.md), [Matrix](https://docs.mapx.at/de/api/matrix.md), [Map Matching](https://docs.mapx.at/de/api/map-matching.md), [Tempolimits](https://docs.mapx.at/de/api/speed-limits.md).

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `heavy` (30/min), `route` (5/s) · Guide: [Navigation](https://docs.mapx.at/de/guides/navigation.md)

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
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
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
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (durch Komma getrennt; `weather_warnings` ergänzt `routes[].weather_warnings`, die Warnungen von GeoSphere Austria, deren Gebiet die Route quert, ohne Geometrie): `legs[].annotation` je Geometriesegment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (nur driving; `legal` ist true bei einem gesetzlichen Tempolimit und false bei einer modellierten Geschwindigkeit wie 53 oder 84, die nicht als Tempolimit angezeigt werden darf), elevation = mittlere Segmenthöhe in m |
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
| `width` | query | string |  | `2.55` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
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
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
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
| `annotations` | query | string |  | – | duration, distance, speed, maxspeed, elevation, weather_warnings (durch Komma getrennt; `weather_warnings` ergänzt `routes[].weather_warnings`, die Warnungen von GeoSphere Austria, deren Gebiet die Route quert, ohne Geometrie): `legs[].annotation` je Geometriesegment; speed in m/s, maxspeed `[{speed, unit: km/h, legal} \| {unknown: true}]` (nur driving; `legal` ist true bei einem gesetzlichen Tempolimit und false bei einer modellierten Geschwindigkeit wie 53 oder 84, die nicht als Tempolimit angezeigt werden darf), elevation = mittlere Segmenthöhe in m |
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
| `width` | query | string |  | `2.55` | Nur driving-truck: Fahrzeugbreite in m (Standard 2.55, 1 bis 3.5) |
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
  "attribution": "mapx.at | GIP.at (CC BY 4.0)"
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `422` | Anfrage gültig, aber nicht beantwortbar (z. B. keine Route) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

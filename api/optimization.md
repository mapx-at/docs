# Tourenplanung

Touren für mehrere Fahrzeuge mit Kapazität, Zeitfenstern und Abhol-/Lieferaufträgen, vereinfacht nach Mapbox Optimization v2.

Verteilt Aufträge auf Fahrzeuge und bringt sie je Fahrzeug in eine günstige Reihenfolge. Die Fahrzeiten kommen aus der Matrix auf dem [GIP.at](https://www.gip.at)-Netz. Für eine einzelne Fahrt mit bester Reihenfolge reicht [Optimierte Reihenfolge](/api/routing/#optimierte-reihenfolge).

**POST https://api.mapx.at/v1/optimization**

Der Rumpf ist JSON (`Content-Type: application/json`, höchstens 256 KB). Zeiten sind Sekunden, Mengen ganze Zahlen (auch mehrere Dimensionen, etwa Gewicht und Volumen), Koordinaten `[lon, lat]`.

## Eingabe

<ParamTable params={[
  { name: "profile", type: "string", default: "driving", desc: "<code>driving</code>, <code>cycling</code> oder <code>walking</code>." },
  { name: "vehicles", type: "array", required: true, desc: "1 bis 10 Fahrzeuge: <code>id</code>, <code>start</code> (Pflicht), <code>end</code> (ohne Angabe endet die Tour am letzten Auftrag), <code>capacity</code> (Liste, ohne Angabe unbegrenzt), <code>time_window</code> (<code>[von, bis]</code>, Standard unbegrenzt ab 0)." },
  { name: "services", type: "array", desc: "Aufträge an einem Ort: <code>id</code>, <code>location</code>, <code>service</code> (Dauer, Standard 0), <code>delivery</code> (Menge, die am Start geladen und hier abgegeben wird, auch <code>amount</code>), <code>pickup</code> (Menge, die hier aufgenommen wird), <code>time_windows</code> (Liste von <code>[von, bis]</code>)." },
  { name: "shipments", type: "array", desc: "Abhol- und Lieferaufträge: <code>id</code>, <code>amount</code>, <code>pickup</code> und <code>delivery</code> mit je <code>location</code>, <code>service</code>, <code>time_windows</code>. Abholung kommt vor der Lieferung, beide im selben Fahrzeug." },
  { name: "options.time_limit_ms", type: "integer", default: "1500", desc: "Rechenzeit der Suche, 50 bis 10000." },
  { name: "options.geometry", type: "boolean", default: "false", desc: "Mit <code>true</code> bekommt jede Tour eine <code>geometry</code> (höchstens 80 Teilstrecken insgesamt); Format über <code>options.geometries</code>." },
]} />

Grenzen: 10 Fahrzeuge, 50 Aufträge (Services plus Sendungen), 60 verschiedene Orte. Ids müssen eindeutig sein.

## Beispiel

<Code lang="bash" code={`curl -X POST https://api.mapx.at/v1/optimization -H 'Content-Type: application/json' -d '{   "profile": "driving",   "vehicles": [     { "id": "melk", "start": [15.3338, 48.2279], "end": [15.3338, 48.2279], "capacity": [3], "time_window": [28800, 64800] },     { "id": "ybbs", "start": [15.0864, 48.1696], "capacity": [3], "time_window": [28800, 64800] }   ],   "services": [     { "id": "poechlarn", "location": [15.2065, 48.2111], "service": 600, "delivery": [1] },     { "id": "loosdorf", "location": [15.3969, 48.2339], "service": 600, "delivery": [1], "time_windows": [[28800, 36000]] },     { "id": "mank", "location": [15.3167, 48.1167], "service": 900, "delivery": [2] }   ],   "shipments": [     { "id": "paket", "amount": [1],       "pickup": { "location": [15.2597, 48.2018], "service": 120 },       "delivery": { "location": [15.0764, 48.1692], "service": 120 } }   ],   "options": { "time_limit_ms": 1500, "geometry": true } }'`} />

Anfrage (lesbar):

<Code lang="json" code={`{
  "profile": "driving",
  "vehicles": [
    { "id": "melk", "start": [15.3338, 48.2279], "end": [15.3338, 48.2279], "capacity": [3], "time_window": [28800, 64800] },
    { "id": "ybbs", "start": [15.0864, 48.1696], "capacity": [3], "time_window": [28800, 64800] }
  ],
  "services": [
    { "id": "poechlarn", "location": [15.2065, 48.2111], "service": 600, "delivery": [1] },
    { "id": "loosdorf", "location": [15.3969, 48.2339], "service": 600, "delivery": [1], "time_windows": [[28800, 36000]] },
    { "id": "mank", "location": [15.3167, 48.1167], "service": 900, "delivery": [2] }
  ],
  "shipments": [
    { "id": "paket", "amount": [1],
      "pickup": { "location": [15.2597, 48.2018], "service": 120 },
      "delivery": { "location": [15.0764, 48.1692], "service": 120 } }
  ],
  "options": { "time_limit_ms": 1500, "geometry": true }
}`} />

## Antwort

<Code lang="json" code={`{
  "code": "Ok",
  "summary": { "cost": 5563.6, "duration": 5563.6, "distance": 104708.3, "service": 3240.0, "waiting_time": 0.0, "routes": 2, "unassigned": 0, "computing_ms": 1517 },
  "routes": [
    { "vehicle": "melk", "duration": 4080.0, "distance": 76800.0, "end_time": 34560.0,
      "steps": [
        { "type": "start", "location": [15.3338, 48.2279], "arrival": 28800.0, "load": [3] },
        { "type": "job", "id": "loosdorf", "location": [15.3969, 48.2339], "arrival": 29280.0, "waiting_time": 0.0, "service": 600.0, "load": [2] },
        { "type": "end", "location": [15.3338, 48.2279], "arrival": 34560.0, "load": [0] }
      ] }
  ],
  "unassigned": []
}`} />

- `summary`: Summen über alle Touren (`cost` = Fahrzeit in Sekunden, `distance` in Metern, `service`, `waiting_time`), Zahl der Touren und der nicht eingeplanten Aufträge, `computing_ms`.
- `routes[]`: eine Tour je genutztem Fahrzeug. `steps[]` hat die Typen `start`, `job`, `pickup`, `delivery`, `end`, dazu `arrival` (Sekunden), `waiting_time`, `service` und `load` nach dem Halt.
- `unassigned[]`: Aufträge ohne Platz, mit `reason` (Kapazität, Erreichbarkeit, Zeitfenster).

## Verfahren und Grenzen

Das Ziel sind zuerst möglichst viele eingeplante Aufträge, dann die kleinste Gesamtfahrzeit. Gerechnet wird mit Regret-Einfügung, lokaler Suche (Umhängen, Umkehren) und Ruin-and-Recreate bis zum Zeitlimit; das Ergebnis ist gut, aber nicht beweisbar optimal. Fahrzeuge starten zu Beginn ihres Zeitfensters und warten bei Bedarf vor einem Zeitfenster; ein späterer Start, der das Warten vermeidet, wird nicht gesucht. Fähigkeiten (Skills), Prioritäten, Pausen und feste Fahrzeugkosten gibt es nicht.

Die Anfrage zählt zu den aufwendigen und hat ein enges Limit. Fehler: `400` ungültige Eingabe (mit Hinweis auf das Feld), `429` Limit, `502` Routing nicht erreichbar. Allgemeines: [Fehler und Limits](/api/fehler/).
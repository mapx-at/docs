# Logistik

Lkw-Routing mit Fahrzeugmaßen, Fahrzeitmatrizen, Tourenplanung und Verkehr für Zustellung und Flotten.

Flotten- und Zustellsoftware braucht eine Route, die ein Lkw wirklich fahren kann, Fahrzeiten für viele Paare, eine gute Reihenfolge der Stopps und die aktuelle Verkehrslage. Alle vier sind je eine Anfrage. Koordinaten sind `lon,lat`, Zeiten in Sekunden, Distanzen in Metern.

Alle Aufrufe brauchen einen Token; `DEIN_TOKEN` durch den eigenen ersetzen ([Zugang und Limits](/de/start/access-and-limits/)). Das SDK setzt ihn einmal mit `mapx.setAccessToken("DEIN_TOKEN")`.

## Lkw-Routen

Das Profil `driving-truck` nimmt die Fahrzeugmaße und meidet Straßen mit passenden Gewichts-, Höhen-, Breiten- und Längenbeschränkungen und Verboten. Ohne Angaben gelten 40 t, 4 m Höhe und 2,55 m Breite.

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/directions/driving-truck/15.6256,48.2049;15.3294,48.2272?weight=40&height=4&width=2.55&length=16.5&truck_type=semi&overview=false" \
  | jq '.routes[0] | {distance, duration, truck, toll_go_km: .toll.truck_go_km}'
```

`routes[0].toll.truck_go_km` ist die GO-mautpflichtige Länge; je Schritt `truck_go_m`. Die Geschwindigkeiten folgen den Lkw-Limits (80 km/h auf Autobahnen). Bezirksweite Verbote aus Freitext-Verordnungen werden nicht geroutet: betroffene Straßen kommen als `steps[].notices[]` mit `type: "truck_restriction"`; der Disposition anzeigen. Für Pkw meiden `exclude=toll` und `exclude=motorway` Mautabschnitte und Autobahnen (nur Profil `driving`).

Mit dem SDK:

```js
const r = await mapx.directions("driving-truck", [[15.6256, 48.2049], [15.3294, 48.2272]], { truck: { weight: 40, height: 4, width: 2.55, length: 16.5, type: "semi" }, steps: true });
const notices = r.routes[0].legs.flatMap((l) => l.steps).flatMap((s) => s.notices ?? []);
```

## Fahrzeiten für viele Paare

`matrix` beantwortet alle Paare in einer Anfrage, im Rahmen der Grenzen der [Matrix-API](/de/api/directions/). Mit `sources` und `destinations` (Indizes) lässt sich nur eine Zeile abfragen.

```js
const depots = [[16.3725, 48.2085], [15.3294, 48.2272], [14.2858, 48.3069]];
const q = depots.map((p) => p.join(",")).join(";");
const m = await (await fetch(`https://api.mapx.at/v1/matrix/driving/${q}?annotations=duration,distance`, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(m.durations[0][1] / 60, "min", m.distances[0][1] / 1000, "km"); // Wien nach Melk
```

## Tour planen

`POST /v1/optimization` teilt Aufträge Fahrzeugen zu und ordnet sie: Kapazität, Zeitfenster, Servicezeiten (Sekunden), Start und Ende je Fahrzeug. Höchstens 10 Fahrzeuge, 50 Aufträge und 60 verschiedene Orte je Anfrage.

```js
const body = {
  profile: "driving",
  vehicles: [{ id: 1, start: [16.3725, 48.2083], end: [16.3725, 48.2083], time_window: [28800, 61200] }],
  services: [
    { id: 1, location: [15.6256, 48.2049], service: 300, time_windows: [[32400, 43200]] },
    { id: 2, location: [14.2858, 48.3069], service: 600 },
  ],
};
const res = await fetch("https://api.mapx.at/v1/optimization", { method: "POST", headers: { "content-type": "application/json", Authorization: "Bearer DEIN_TOKEN" }, body: JSON.stringify(body) });
const tour = await res.json();
console.log(tour.routes[0].steps.map((s) => s.id ?? s.type), tour.unassigned); // unassigned: Aufträge, die nicht passten
```

Das Ergebnis mit einer `directions`-Anfrage durch die Orte aus `tour.routes[0].steps` zeichnen (bis zu 25 Punkte); siehe das [Beispiel Tourenplanung](/de/examples/optimization/).

## Verkehr

`driving-traffic` liefert eine Route mit aktuellem Verkehr und Sperren. Meldungen auf der Route kommen mit `incidents=true` in `legs[].incidents`. Für eine Region die aktiven Meldungen auflisten oder den Verkehrsstil zeigen:

```js
const bbox = [14.5, 47.8, 16.9, 48.6]; // Niederösterreich
const { features } = await (await fetch(`https://api.mapx.at/v1/traffic/incidents?bbox=${bbox}&active=true&type=construction&limit=50`, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(features.length, features[0]?.properties.road, features[0]?.properties.text);
```

Der Kartenstil `traffic-day` zeigt die Verkehrslage; siehe das [Beispiel](/de/examples/traffic/). Verkehrsdaten: ASFINAG / EVIS.AT (CC BY 4.0).

## Nächste Schritte

- [Navigation](/de/guides/navigation/): alle Profile und Parameter
- [Verkehr](/de/guides/traffic/)
- [Fehler und Limits](/de/api/errors/): Rate-Limit-Zonen und was bei `429` zu tun ist
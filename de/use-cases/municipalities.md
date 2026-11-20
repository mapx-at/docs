# Karten für Gemeinden und Verwaltung

Source: https://docs.mapx.at/de/use-cases/municipalities/

> Adressvalidierung, ÖV-Abfahrten und eine eingebettete Karte für Gemeinde-Websites und Verwaltungswerkzeuge.

Gemeindewerkzeuge arbeiten mit Adressen, müssen für Bürgerinnen und Bürger verständlich sein und sollen keine Lizenz brauchen. Die Bausteine unten verbinden Adressregister, Fahrplan und Karte für die Website einer Gemeinde.

Alle Aufrufe brauchen einen Token; `DEIN_TOKEN` durch den eigenen ersetzen ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)).

## Adressen prüfen

Formulare für Anmeldungen, Anträge und Meldungen sollen nur existierende Adressen annehmen. `address/validate` gleicht die Eingabe mit dem Adressregister ab und liefert `match`, `corrected`, `ambiguous` oder `not_found` mit der korrigierten Adresse und Koordinaten.

```js
const q = new URLSearchParams({ street: "Haubtplatz", housenumber: "1", postcode: "3390", place: "Melk" });
const r = await (await fetch(`https://api.mapx.at/v1/address/validate?${q}`, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(r.status, r.deviations.map((d) => `${d.field}: ${d.input} > ${d.expected}`), r.address?.full_address ?? r.candidates?.map((c) => c.full_address));
```

Während der Eingabe das [Adress-Autofill](https://docs.mapx.at/de/examples/autofill.md) anbieten; für Listen mit vielen Adressen `POST /v1/geocode/batch` ([Suche](https://docs.mapx.at/de/guides/search.md)), nur mit geheimem Token (`mx_sk_…`) vom Server, sonst `403 pk-not-allowed`.

## Abfahrten am Gemeindeamt

Bürgerinnen und Bürger fragen „Wann fährt der nächste Bus?“. Der Fahrplan (Soll, keine Echtzeitdaten) ist eine Anfrage entfernt:

```js
const H = { headers: { Authorization: "Bearer DEIN_TOKEN" } };
const { stops: [stop] } = await (await fetch("https://api.mapx.at/v1/stops?near=15.3294,48.2272&radius=400&limit=1", H)).json();
const { departures } = await (await fetch(`https://api.mapx.at/v1/departures?stop=${encodeURIComponent(stop.id)}&limit=6`, H)).json();
document.querySelector("#abfahrten").replaceChildren(...departures.map((d) => Object.assign(document.createElement("li"), { textContent: `${d.time.slice(11, 16)} ${d.line.name} → ${d.headsign}` })));
```

Auf einer Karte per Klick zeigt das [Beispiel](https://docs.mapx.at/de/examples/departures.md) dasselbe. Siehe [ÖV-Abfahrten](https://docs.mapx.at/de/guides/transit.md). Fahrplandaten: ÖBB-Personenverkehr AG, Datenquelle Mobilitätsverbünde Österreich (CC BY 4.0).

## Karte in die Gemeinde-Website einbetten

Eine Karte braucht eine HTML-Datei, MapLibre und die Style-URL, einen kostenlosen Token und keinen Build ([Karte einbetten](https://docs.mapx.at/de/examples/embed-map.md)). Die Seite im Content-Management-System ablegen oder die Datei als `<iframe>` einbinden:

```html
<iframe src="/karte.html" title="Karte unserer Gemeinde" style="width:100%;height:480px;border:0" loading="lazy"></iframe>
```

Für ein einfaches Bild ohne JavaScript (Druck, Newsletter, Seite ohne Skripte) ein [statisches Bild](https://docs.mapx.at/de/guides/static-images.md) nehmen:

```html
<img alt="Lage des Gemeindeamts" width="600" height="400"
     src="https://api.mapx.at/v1/static/streets/pin-l%2Be11d48(15.3294,48.2272)/15.3294,48.2272,16/600x400.png?access_token=DEIN_TOKEN">
```

Eigene Daten (Spielplätze, Sammelstellen, Sperren) als [GeoJSON auf der Karte](https://docs.mapx.at/de/examples/geojson-overlay.md) zeigen. Die `<img>`-URL enthält den Token: einen öffentlichen Token (`mx_pk_…`) mit der eigenen Website als erlaubter Origin verwenden; Bilder für E-Mail oder PDF auf dem Server mit einem geheimen Token (`mx_sk_…`) erzeugen. Die Attribution der Daten sichtbar lassen ([Attribution](https://docs.mapx.at/de/reference/attribution.md)).

## Nächste Schritte

- [Suche](https://docs.mapx.at/de/guides/search.md)
- [ÖV-Abfahrten](https://docs.mapx.at/de/guides/transit.md)
- [Basiskarte stylen](https://docs.mapx.at/de/guides/style-the-basemap.md) in den Farben der Gemeinde

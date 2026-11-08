# Gemeinden

Grundstücke, Adressvalidierung, ÖV-Abfahrten und eine eingebettete Karte für Gemeinde-Websites und Verwaltungswerkzeuge.

Gemeindewerkzeuge arbeiten mit Grundstücken und Adressen, müssen für Bürgerinnen und Bürger verständlich sein und sollen keine Lizenz brauchen. Die Bausteine unten verbinden Kataster, Adressregister, Fahrplan und Karte für die Website einer Gemeinde.

## Grundstücke

Der Kataster liegt als Vektor-Tileset zum Zeichnen und als API für das Grundstück an einem Punkt vor.

```js
const map = mapx.map({ container: "map", style: "satellite-streets", center: [15.3294, 48.2272], zoom: 17 });
new mapx.ParcelPopup(map); // Klick: Nummer, Fläche, Katastralgemeinde, Gebäude
```

Ein Grundstück über seine Nummer suchen (die Katastralgemeinde gehört zur Identität):

```bash
curl -s "https://api.mapx.at/v1/geocode?q=KG%2014143%20Gst%20473/1&types=parcel&limit=1" | jq '.features[0] | {place_name, center, id: .properties.parcel_id}'
```

Die Grenzen stammen aus der Katastralmappe des BEV (DKM). Die Fläche wird aus den Grenzlinien berechnet und weicht um Promille vom Register ab. Dort, wo die Daten gezeigt werden, darauf hinweisen: zur Orientierung, nicht rechtsverbindlich.

## Adressen prüfen

Formulare für Anmeldungen, Anträge und Meldungen sollen nur existierende Adressen annehmen. `address/validate` gleicht die Eingabe mit dem Adressregister ab und liefert `match`, `corrected`, `ambiguous` oder `not_found` mit der korrigierten Adresse und Koordinaten.

```js
const q = new URLSearchParams({ street: "Haubtplatz", housenumber: "1", postcode: "3390", place: "Melk" });
const r = await (await fetch(`https://api.mapx.at/v1/address/validate?${q}`)).json();
console.log(r.status, r.deviations.map((d) => `${d.field}: ${d.input} > ${d.expected}`), r.address?.full_address ?? r.candidates?.map((c) => c.full_address));
```

Während der Eingabe das [Adress-Autofill](/de/examples/autofill/) anbieten; für Listen mit vielen Adressen `POST /v1/geocode/batch` ([Suche](/de/guides/search/)).

## Abfahrten am Gemeindeamt

Bürgerinnen und Bürger fragen „Wann fährt der nächste Bus?“. Der Fahrplan (Soll, keine Echtzeitdaten) ist eine Anfrage entfernt:

```js
const [stop] = await mapx.stops({ near: [15.3294, 48.2272], radius: 400, limit: 1 });
const board = await mapx.departures(stop.id, { limit: 6 });
document.querySelector("#abfahrten").innerHTML = board.map((d) => `<li>${d.time.slice(11, 16)} ${d.line.name} → ${d.headsign}</li>`).join("");
```

Oder `new mapx.DeparturesPopup(map)` an eine Karte hängen. Siehe [ÖV-Abfahrten](/de/guides/transit/) und das [Beispiel](/de/examples/departures/). Fahrplandaten: ÖBB-Personenverkehr AG, Datenquelle Mobilitätsverbünde Österreich (CC BY 4.0).

## Karte in die Gemeinde-Website einbetten

Eine Karte braucht eine HTML-Datei, MapLibre und die Style-URL, einen kostenlosen Token und keinen Build ([Karte einbetten](/de/examples/embed-map/)). Die Seite im Content-Management-System ablegen oder die Datei als `<iframe>` einbinden:

```html
<iframe src="/karte.html" title="Karte unserer Gemeinde" style="width:100%;height:480px;border:0" loading="lazy"></iframe>
```

Für ein einfaches Bild ohne JavaScript (Druck, Newsletter, Seite ohne Skripte) ein [statisches Bild](/de/guides/static-images/) nehmen:

```html
<img alt="Lage des Gemeindeamts" width="600" height="400"
     src="https://api.mapx.at/v1/static/streets/pin-l%2Be11d48(15.3294,48.2272)/15.3294,48.2272,16/600x400.png">
```

Eigene Daten (Spielplätze, Sammelstellen, Sperren) als [GeoJSON auf der Karte](/de/examples/geojson-overlay/) zeigen. Die Attribution der Daten sichtbar lassen ([Attribution](/de/reference/attribution/)).

## Nächste Schritte

- [Grundstücke](/de/guides/parcels/) und [Suche](/de/guides/search/)
- [ÖV-Abfahrten](/de/guides/transit/)
- [Basiskarte stylen](/de/guides/style-the-basemap/) in den Farben der Gemeinde
# Suche

Geocoding, Type-ahead-Suche, Adress-Autofill und Validierung.

Den Endpunkt nach der Aktion der Nutzer wählen.

| Aufgabe | Endpunkt |
|---|---|
| Eine eingetippte Adresse oder einen Ort in Koordinaten umwandeln | [`/v1/geocode`](/de/api/geocoding/) |
| Vorschläge während der Eingabe zeigen | [`/v1/searchbox/suggest`](/de/api/search-box/) |
| Ein Postadress-Formular ausfüllen | [`/v1/autofill/suggest`](/de/api/address/) |
| Eine Adresse prüfen | [`/v1/address/validate`](/de/api/address/) |
| Die Adresse an einem Punkt finden | [`/v1/reverse`](/de/api/geocoding/) |
| Orte einer Art zeigen (Apotheke, Bahnhof) | [`/v1/searchbox/category/{category}`](/de/api/search-box/) |

Alle Daten sind amtlich: Adressregister, Ortsnamen, Verwaltungsgrenzen und Kataster vom BEV. Anfragen liegen in der Zone `search`, 10 pro Sekunde.

## Eine Adresse geocodieren

```js
const url = "https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&types=address&limit=3";
const { features } = await (await fetch(url)).json();
const [lon, lat] = features[0].center;
```

Immer `types` angeben (`address`, `street`, `locality`, `place`, `district`, `region`, `postcode`, `poi`). Das Ergebnis lässt sich mit `bbox=minlon,minlat,maxlon,maxlat` eingrenzen oder mit `proximity=lon,lat` nach Entfernung ordnen.

## Reverse Geocoding

```bash
curl -s "https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&types=address&limit=1" | jq '.features[0].place_name'
```

## Type-ahead-Suche

`suggest` bei jeder Tipppause aufrufen (Debounce 200 ms), danach `retrieve` einmal für den gewählten Eintrag. Für beide Aufrufe denselben `session_token` senden.

```js
const token = crypto.randomUUID();
const s = await (await fetch(`https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=5&session_token=${token}`)).json();
const id = s.suggestions[0].mapbox_id;
const r = await (await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${id}?session_token=${token}`)).json();
const [lon, lat] = r.features[0].geometry.coordinates;
```

Vorschläge tragen eine Kategorie (`poi_category_ids`, zum Beispiel `place_of_worship_church`) und den Verwaltungskontext. Die vorige Anfrage abbrechen, wenn eine neue Taste gedrückt wird (`AbortController`). Beispiel: [Suche](/de/examples/search/).

## Orte nach Kategorie

```bash
curl -s "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq ".listItems[0:3]"
curl -s "https://api.mapx.at/v1/searchbox/category/pharmacy?proximity=16.3725,48.2085&limit=5"
```

`list/category` liefert alle Kategorie-IDs mit lokalisierten Namen (`language=de` oder `en`).

## Adressformulare

Autofill liefert getrennte Felder, die sich auf Formularfelder abbilden lassen.

```js
const a = await (await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3")).json();
const { address_line1, postcode, place, coordinates } = a.suggestions[0];
```

Die Validierung prüft, was der Nutzer eingegeben hat:

```bash
curl -s "https://api.mapx.at/v1/address/validate?q=Hauptstr.%2012a,%203390" | jq '{status, confidence, address}'
```

`status` ist `match`, `corrected`, `ambiguous` (dann `candidates[]`) oder `not_found`. Abkürzungen, Umlaute und `ß` werden verstanden. Für strukturierte Eingaben POST mit JSON verwenden.

## Batch

`POST /v1/geocode/batch` geocodiert bis zu 1000 Anfragen in einem Request (Zone `heavy`, 30 pro Minute).

## Nächste Schritte

- [Geocoding API](/de/api/geocoding/), [Search Box API](/de/api/search-box/), [Address API](/de/api/address/)
- [Grundstücke](/de/guides/parcels/): Suchergebnisse vom Typ parcel
- [Fehler und Limits](/de/api/errors/)
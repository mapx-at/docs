# Adresssuche: Geocoding und Autofill

Source: https://docs.mapx.at/de/guides/search/

> Adress- und Ortssuche für Österreich in die eigene App einbauen: Geocoding, Search Box mit Vorschlägen beim Tippen, Adress-Autofill und Adressvalidierung.

Den Endpunkt nach der Aktion der Nutzer wählen.

| Aufgabe | Endpunkt |
|---|---|
| Eine eingetippte Adresse oder einen Ort in Koordinaten umwandeln | [`/v1/geocode`](https://docs.mapx.at/de/api/geocoding.md) |
| Vorschläge während der Eingabe zeigen | [`/v1/searchbox/suggest`](https://docs.mapx.at/de/api/search-box.md) |
| Ein Postadress-Formular ausfüllen | [`/v1/autofill/suggest`](https://docs.mapx.at/de/api/address.md) |
| Eine Adresse prüfen | [`/v1/address/validate`](https://docs.mapx.at/de/api/address.md) |
| Die Adresse an einem Punkt finden | [`/v1/reverse`](https://docs.mapx.at/de/api/geocoding.md) |
| Orte einer Art zeigen (Tankstelle, Bahnhof) | [`/v1/searchbox/category/{category}`](https://docs.mapx.at/de/api/search-box.md) |

Alle Endpunkte brauchen einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Beispiele senden ihn im Header `Authorization`, den auch Browser senden dürfen. Alle Daten sind offene Verwaltungsdaten: Adressregister, Ortsnamen, Verwaltungsgrenzen vom BEV. Anfragen liegen in der Zone `search`, 10 pro Sekunde.

## Eine Adresse geocodieren

```js
const url = "https://api.mapx.at/v1/geocode?q=Herrengasse%2013,%201010%20Wien&types=address&limit=3";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
const [lon, lat] = features[0].geometry.coordinates;
```

`features` kann leer sein; vor dem Zugriff auf `features[0]` die Länge prüfen. Immer `types` angeben (`address`, `street`, `locality`, `place`, `district`, `region`, `postcode`, `poi`, `road_ref` für Straßenkilometer wie `A1 km 41`). Das Ergebnis lässt sich mit `bbox=minlon,minlat,maxlon,maxlat` eingrenzen oder mit `proximity=lon,lat` nach Entfernung ordnen.

Alle Such-Endpunkte (geocode, reverse, batch, Search Box, Autofill) antworten im selben Format: eine GeoJSON-`FeatureCollection`, jeder Treffer mit stabiler `id`, Punkt-`geometry` und in `properties` den Feldern `type`, `name`, `place_formatted` (Anzeigezeile: `name, place_formatted`), `relevance`, `accuracy` und `address`. PLZ, Ort, Straße und Hausnummer stehen getrennt in `properties.address`. Ein Ort nach der Hausnummer hat Vorrang: `Hauptplatz 12a Graz` liefert Graz (Hauptplatz 12, die Straße, die Nachbarnummern), auch wenn es 12a nur anderswo gibt. Koordinaten wie `48.21, 16.37` als Suchtext werden wie bei Reverse Geocoding beantwortet.

## Reverse Geocoding

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&types=address&limit=1" | jq '.features[0].properties | {name, place_formatted, distance}'
```

## Type-ahead-Suche

`suggest` bei jeder Tipppause aufrufen (Debounce 200 ms). Die Vorschläge tragen schon die Koordinaten; `retrieve` liefert einen einzelnen Treffer über seine `id` erneut (etwa einen früher gespeicherten). `session_token` ist optional und ohne Wirkung.

```js
const token = crypto.randomUUID();
const s = await (await fetch(`https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=5&session_token=${token}`, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
const id = s.features[0].id;
const r = await (await fetch(`https://api.mapx.at/v1/searchbox/retrieve/${id}?session_token=${token}`, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
const [lon, lat] = r.features[0].geometry.coordinates;
```

POIs tragen eine Kategorie (`properties.category`, zum Beispiel `place_of_worship_church`) und die Verwaltungsangaben in `properties.address`. Die vorige Anfrage abbrechen, wenn eine neue Taste gedrückt wird (`AbortController`). Beispiel: [Suche](https://docs.mapx.at/de/examples/search.md).

## Orte nach Kategorie

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq ".listItems[0:3]"
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/searchbox/category/fuel?proximity=16.3725,48.2085&limit=5"
```

`list/category` liefert alle Kategorie-IDs mit lokalisierten Namen (`language=de` oder `en`).

## Adressformulare

Autofill liefert nur Adressen, mit getrennten Feldern in `properties.address`, die sich auf Formularfelder abbilden lassen.

```js
const a = await (await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013,%201010%20Wien&limit=3", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
const { name, address } = a.features[0].properties; // address: street, housenumber, postcode, place, …
const [lon, lat] = a.features[0].geometry.coordinates;
```

Die Validierung prüft, was der Nutzer eingegeben hat:

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk" | jq '{status, confidence, address}'
```

`status` ist `match`, `corrected`, `ambiguous` (dann `candidates[]`) oder `not_found`. Gibt es die Hausnummer in der Straße nicht, kommt `not_found` mit den nächsten Nummern in `candidates[]`. Abkürzungen, Umlaute und `ß` werden verstanden. Für strukturierte Eingaben POST mit JSON verwenden.

## Batch

`POST /v1/geocode/batch` geocodiert bis zu 1000 Anfragen in einem Request (Zone `heavy`, 30 pro Minute). Batch-Geocoding braucht einen geheimen Token (`mx_sk_…`) und gehört deshalb auf den Server; mit einem öffentlichen Token antwortet die API mit `403` `pk-not-allowed`. Ein ungültiger Eintrag steht als Problem-Objekt (`status` 400, `code`, `param`) an seiner Stelle im Ergebnis, die übrigen werden beantwortet.

## Nächste Schritte

- [Geocoding API](https://docs.mapx.at/de/api/geocoding.md), [Search Box API](https://docs.mapx.at/de/api/search-box.md), [Address API](https://docs.mapx.at/de/api/address.md)
- [Fehler und Limits](https://docs.mapx.at/de/api/errors.md)

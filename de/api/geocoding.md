# Geocoding API Österreich

Source: https://docs.mapx.at/de/api/geocoding/

> Geocoding-API für Österreich: Vorwärts-, Rückwärts- und Batch-Geocoding von Adressen, Straßen, Orten und Postleitzahlen, mit Beispielen.

Wandelt Adresse oder Ortsname in Koordinaten um und Koordinaten in die nächste Adresse.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `heavy` (30/min) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Geocoding {#forward-geocoding}

`GET /v1/geocode`

Suche über das Adressregister des BEV, Straßen, Orte, Gemeinden, Bezirke, Postleitzahlen und Orte von Interesse. Straßenkilometer: `B83 km 329,7`, `A1 km 41` oder `L190 km 3.5` (Klasse A, S, B oder L, Nummer, das Wort `km`) liefern den Punkt der Straße bei diesem Kilometer als `properties.type` `road_ref` mit `properties.road_ref` (`ref`, `km`); der Index enthält auf Autobahnen, Schnellstraßen und Bundesstraßen alle 0,1 km einen Punkt und auf Landesstraßen alle 0,5 km (GIP.at), geliefert wird der nächstgelegene; dieselbe Landesstraßennummer in mehreren Bundesländern ergibt mehrere Treffer.

Adressen: nennt die Eingabe nach der Hausnummer einen Ort oder eine PLZ (`Hauptplatz 12a Graz`, `Rathausplatz 1, 3100`), kommen Treffer in diesem Ort zuerst; gibt es die Nummer dort nicht, folgen die Grundnummer (`12` zu `12a`), die Straße in diesem Ort und die nächsten Nummern, nie zuerst dieselbe Nummer in einem anderen Ort. Hausnummernbereiche (`1-3`, `15 - 16`) bleiben Bereiche. Häufige Abkürzungen werden ausgeschrieben (`Hptpl.`, `Str.`, `Hauptstr.`, `Rathauspl.`, `Pl.`, `G.`, `Bhf`). Koordinaten als Suchtext (`16.37,48.21` oder `48.21, 16.37`, beide mit Dezimalpunkt; Reihenfolge aus dem Wertebereich Österreichs erkannt) antworten wie `/v1/reverse` mit `query` [lng, lat]. Antwort: FeatureCollection im gemeinsamen Format aller Such-Endpunkte (Schema `Place`): `id`, Punkt-`geometry`, `bbox` bei Flächen, und in `properties` immer `type`, `name`, `place_formatted`, `relevance` (Trefferwert der Suche, 0 bis 1, niedriger beim Rückfall auf Grundnummer, Straße oder Nachbarnummer), `accuracy` (`rooftop` = Punkt am Gebäude, `parcel` = Punkt am Grundstück, `street`, sonst `null`) und `address` (die Felder einzeln: `street`, `housenumber`, `postcode`, `place`, `locality`, `municipality`, `district`, `state`, nur vorhandene); mit `proximity` auch `distance` (m). Gewässer (Seen, Flüsse, Bäche aus den BEV-Geonamen) sind Treffer vom Typ `place` mit `properties.class` `see`, `fluss`, `bach` oder `gewaesser`. `query` listet die Suchwörter ohne Satzzeichen. Für günstige Abfragen immer `types` angeben.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string | ja | – | Suchtext (auch Pfadform `/v1/geocode/{q}.json`) |
| `limit` | query | integer (1–10) |  | `5` | 1 bis 10, Standard 5 |
| `types` | query | string |  | – | Ergebnistypen, durch Komma getrennt: address, street, locality, place, district, region, postcode, poi, road_ref (Straßenkilometer; Eingaben wie `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lng -180 bis 180, lat -90 bis 90; sonst 400 `invalid-parameter`) |
| `proximity` | query | string |  | – | lng,lat; nahe Treffer (bis etwa 40 km) stehen vor entfernten, die nur etwas besser passen, Treffer jenseits von 40 km ohne besondere Bedeutung nie vor nahen; exakte Ortsnamen bleiben oben; nennt der Suchtext eine Kategorie (`Bahnhof`, `Museum`, siehe `/v1/searchbox/list/category`), kommen auch die nächsten Orte dieser Kategorie; wird ignoriert, wenn bbox gesetzt ist |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&limit=3&types=address" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&limit=3&types=address", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode?q=Herrengasse%2013%20Wien&limit=3&types=address", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
{
  "type": "FeatureCollection",
  "query": [
    "Herrengasse",
    "13"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-6792483",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.365282,
          48.210154
        ]
      },
      "properties": {
        "type": "address",
        "name": "Herrengasse 13",
        "place_formatted": "Innere Stadt, 1010 Wien, W",
        "relevance": 0.97,
        "accuracy": "parcel",
        "address": {
          "street": "Herrengasse",
          "housenumber": "13",
          "postcode": "1010",
          "place": "Wien",
          "locality": "Innere Stadt",
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "zoom": 18
      }
    }
  ],
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `query` | array | Nur Geocoding: die Suchwörter, bzw. [lng, lat] bei Rückwärtssuche und Koordinaten als Suchtext. |
| `features` | array | Treffer, der beste bzw. nächste zuerst. Jeder ist ein GeoJSON-Punkt-Feature mit stabiler `id` (und `bbox` bei Flächen); `properties` enthalten immer `type`, `name`, `place_formatted`, `relevance`, `accuracy` und `address`, je nach Treffer auch `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (Schema `Place`). |
| `warning` | string | Nur Rückwärtssuche: der Punkt liegt außerhalb Österreichs. |
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe verpflichtend. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Geocoding, Pfadform {#forward-geocoding-path-form}

`GET /v1/geocode/{q}`

Wie `/v1/geocode?q=...`; der Suchtext ist das Pfadsegment (optional mit `.json`).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | path | string | ja | – | Suchtext, URL-codiert |
| `limit` | query | string |  | `5` | 1 bis 10, Standard 5 |
| `types` | query | string |  | – | Ergebnistypen, durch Komma getrennt: address, street, locality, place, district, region, postcode, poi, road_ref (Straßenkilometer; Eingaben wie `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lng -180 bis 180, lat -90 bis 90; sonst 400 `invalid-parameter`) |
| `proximity` | query | string |  | – | lng,lat |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/geocode/Stephansplatz%20Wien.json?limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode/Stephansplatz%20Wien.json?limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode/Stephansplatz%20Wien.json?limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "query": [
    "Stephansplatz",
    "Wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "station-at_49_1320",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.371607,
          48.208086
        ]
      },
      "properties": {
        "type": "poi",
        "name": "Wien Stephansplatz",
        "place_formatted": "Bahnhof, Wien",
        "relevance": 0.999,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "zoom": 15,
        "class": "station",
        "category": "railway_station",
        "maki": "rail",
        "stop_id": "at:49:1320",
        "modes": [
          "subway"
        ]
      }
    }
  ],
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `query` | array | Nur Geocoding: die Suchwörter, bzw. [lng, lat] bei Rückwärtssuche und Koordinaten als Suchtext. |
| `features` | array | Treffer, der beste bzw. nächste zuerst. Jeder ist ein GeoJSON-Punkt-Feature mit stabiler `id` (und `bbox` bei Flächen); `properties` enthalten immer `type`, `name`, `place_formatted`, `relevance`, `accuracy` und `address`, je nach Treffer auch `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (Schema `Place`). |
| `warning` | string | Nur Rückwärtssuche: der Punkt liegt außerhalb Österreichs. |
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe verpflichtend. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Reverse Geocoding (nächste Adresse, Straße, Ort) {#reverse-geocoding-nearest-address-street-place}

`GET /v1/reverse`

Nächste Objekte um einen Punkt, das nächste zuerst, im selben Format wie geocode (Schema `Place`), jeweils mit `properties.distance` (Meter vom Punkt); `relevance` ist 1. Mit `types` einschränken (z. B. `address`). Eine Koordinate, die keine Zahl ist oder außerhalb des Wertebereichs liegt, ergibt 400 `invalid-coordinates` mit `param` `lng` bzw.

`lat`; `limit` und `radius` werden getrennt geprüft (400 `invalid-parameter`). Für einen Punkt außerhalb Österreichs hat die Antwort keine Features und einen Hinweis `warning` (oft ist die Reihenfolge vertauscht; sie lautet lng, lat).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lng` | query | string | ja | – | Länge |
| `lat` | query | string | ja | – | Breite |
| `radius` | query | string |  | `500` | Meter, 1 bis 5000, Standard 500 |
| `limit` | query | string |  | `1` | 1 bis 10, Standard 1 |
| `types` | query | string |  | – | Wie bei geocode |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&limit=1&types=address" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&limit=1&types=address", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/reverse?lng=16.3725&lat=48.2085&limit=1&types=address", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "query": [
    16.3725,
    48.2085
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-6793782",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.372605,
          48.208728
        ]
      },
      "properties": {
        "type": "address",
        "name": "Stephansplatz 1",
        "place_formatted": "Innere Stadt, 1010 Wien, W",
        "relevance": 1,
        "accuracy": "parcel",
        "address": {
          "street": "Stephansplatz",
          "housenumber": "1",
          "postcode": "1010",
          "place": "Wien",
          "locality": "Innere Stadt",
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "distance": 27,
        "zoom": 18
      }
    }
  ],
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `query` | array | Nur Geocoding: die Suchwörter, bzw. [lng, lat] bei Rückwärtssuche und Koordinaten als Suchtext. |
| `features` | array | Treffer, der beste bzw. nächste zuerst. Jeder ist ein GeoJSON-Punkt-Feature mit stabiler `id` (und `bbox` bei Flächen); `properties` enthalten immer `type`, `name`, `place_formatted`, `relevance`, `accuracy` und `address`, je nach Treffer auch `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (Schema `Place`). |
| `warning` | string | Nur Rückwärtssuche: der Punkt liegt außerhalb Österreichs. |
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe verpflichtend. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Batch-Geocoding (bis zu 1000 Anfragen) {#batch-geocoding-up-to-1000-requests}

`POST /v1/geocode/batch`

JSON-Array; jeder Eintrag hat `q` oder strukturiert `street`/`housenumber`/`postcode`/`place`, optional `types` (String oder Liste), `limit` (1 bis 10), `proximity` [lng,lat], `bbox`. Antwort: Array in Eingabereihenfolge mit einer FeatureCollection je Eintrag (gleiches Format wie geocode, Schema `Place`, samt Ortsvorrang und Koordinaten als `q`; ohne `distance`); ein ungültiger Eintrag lässt den Batch nicht scheitern, sondern wird an seiner Stelle mit einem Problem-Objekt beantwortet (RFC 9457: `type`, `title`, `status` 400, `detail`, `code`, `param`).

Braucht einen geheimen Token (`mx_sk_`); öffentliche Tokens erhalten `403 pk-not-allowed`, im Playground daher nicht ausführbar.

Ein fehlerhafter Body (kein Array, leer, mehr als 1000 Einträge) ergibt weiterhin 400 für den ganzen Aufruf. `proximity` reiht hier nur gleich gute Treffer (keine eigene Umkreissuche wie bei geocode). Die Einträge werden in Teilen gesucht, höchstens 4 parallel; der ganze Batch zählt im Rate-Limit als eine Anfrage. Höchstens 512 KB.

**Request-Body (JSON)**

```json
[
  {
    "q": "Herrengasse 13 Wien",
    "types": "address",
    "limit": 1
  },
  {
    "street": "Hauptplatz",
    "housenumber": "3",
    "postcode": "3390"
  }
]
```

**Beispiel**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/geocode/batch" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '[{"q":"Herrengasse 13 Wien","types":"address","limit":1},{"street":"Hauptplatz","housenumber":"3","postcode":"3390"}]' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode/batch", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify([
    {
      "q": "Herrengasse 13 Wien",
      "types": "address",
      "limit": 1
    },
    {
      "street": "Hauptplatz",
      "housenumber": "3",
      "postcode": "3390"
    }
  ]),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/geocode/batch", headers={"Authorization": "Bearer YOUR_TOKEN"}, json=[
    {
      "q": "Herrengasse 13 Wien",
      "types": "address",
      "limit": 1
    },
    {
      "street": "Hauptplatz",
      "housenumber": "3",
      "postcode": "3390"
    }
  ])
data = res.json()
```

**Beispielantwort** (gekürzt: Listen zeigen nur die ersten Einträge)

```json
[
  {
    "type": "FeatureCollection",
    "query": [
      "Herrengasse",
      "13"
    ],
    "features": [
      {
        "type": "Feature",
        "id": "address-6792483",
        "geometry": {
          "type": "Point",
          "coordinates": [
            16.365282,
            48.210154
          ]
        },
        "properties": {
          "type": "address",
          "name": "Herrengasse 13",
          "place_formatted": "Innere Stadt, 1010 Wien, W",
          "relevance": 0.97,
          "accuracy": "parcel",
          "address": {
            "street": "Herrengasse",
            "housenumber": "13",
            "postcode": "1010",
            "place": "Wien",
            "locality": "Innere Stadt",
            "municipality": "Wien",
            "district": "Wien (Stadt)",
            "state": "Wien"
          },
          "zoom": 18
        }
      }
    ],
    "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)"
  },
  {
    "type": "https://docs.mapx.at/api/errors/#invalid-parameter",
    "title": "Invalid parameter",
    "status": 400,
    "detail": "types: unknown value 'parcel' (allowed: address, street, locality, place, district, region, postcode, poi, road_ref)",
    "code": "invalid-parameter",
    "param": "types",
    "allowed": [
      "address",
      "street"
    ]
  }
]
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt, ist ungültig oder unbekannt (`unknown-parameter`) |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `405` | Methode nicht erlaubt |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

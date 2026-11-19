# Geocoding API Österreich

Source: https://docs.mapx.at/de/api/geocoding/

> Geocoding-API für Österreich: Vorwärts-, Rückwärts- und Batch-Geocoding von Adressen, Straßen, Orten, Postleitzahlen und Grundstücken, mit Beispielen.

Wandelt Adresse oder Ortsname in Koordinaten um und Koordinaten in die nächste Adresse.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s), `heavy` (30/min) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Geocoding {#forward-geocoding}

`GET /v1/geocode`

Suche über das amtliche Adressregister, Straßen, Orte, Gemeinden, Bezirke, Postleitzahlen und Grundstücke. Straßenkilometer: `B83 km 329,7`, `A1 km 41` oder `L190 km 3.5` (Klasse A, S, B oder L, Nummer, das Wort `km`) liefern den Punkt der Straße bei diesem Kilometer als `place_type` `road_ref` mit `properties.road_ref` (`ref`, `km`); der Index enthält auf Autobahnen, Schnellstraßen und Bundesstraßen alle 0,1 km einen Punkt und auf Landesstraßen alle 0,5 km (GIP.at), geliefert wird der nächstgelegene; dieselbe Landesstraßennummer in mehreren Bundesländern ergibt mehrere Treffer.

Antwort: FeatureCollection mit `place_type`, `place_name`, `center`, `geometry`, `bbox`. Für günstige Abfragen immer `types` angeben.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string | ja | – | Suchtext (auch Pfadform `/v1/geocode/{q}.json`) |
| `limit` | query | integer (1–10) |  | `5` | 1 bis 10, Standard 5 |
| `types` | query | string |  | – | Ergebnistypen, durch Komma getrennt: address, street, locality, place, district, region, postcode, poi, parcel (nur Grundstücke; Eingaben wie `KG 14143 Gst 473/1`, `14143/473/1`, `Landeck .102`), road_ref (Straßenkilometer; Eingaben wie `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `proximity` | query | string |  | – | lon,lat; nahe Treffer (bis etwa 40 km) stehen vor entfernten, die nur etwas besser passen; exakte Ortsnamen bleiben oben; wird ignoriert, wenn bbox gesetzt ist |

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

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "query": [
    "herrengasse",
    "13",
    "wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-1234567",
      "place_type": [
        "address"
      ],
      "place_name": "Herrengasse 13, 1010 Wien",
      "text": "Herrengasse 13",
      "center": [
        16.3665,
        48.2096
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3665,
          48.2096
        ]
      },
      "properties": {}
    }
  …
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Geocoding, Mapbox-Pfadform {#forward-geocoding-mapbox-path-form}

`GET /v1/geocode/{q}`

Wie `/v1/geocode?q=...`; der Suchtext ist das Pfadsegment (optional mit `.json`).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | path | string | ja | – | Suchtext, URL-codiert |
| `limit` | query | string |  | `5` | 1 bis 10, Standard 5 |
| `types` | query | string |  | – | Ergebnistypen, durch Komma getrennt: address, street, locality, place, district, region, postcode, poi, parcel (nur Grundstücke; Eingaben wie `KG 14143 Gst 473/1`, `14143/473/1`, `Landeck .102`), road_ref (Straßenkilometer; Eingaben wie `B83 km 329,7`, `A1 km 41`, `L190 km 3.5`) |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
| `proximity` | query | string |  | – | lon,lat |

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
    "herrengasse",
    "13",
    "wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-1234567",
      "place_type": [
        "address"
      ],
      "place_name": "Herrengasse 13, 1010 Wien",
      "text": "Herrengasse 13",
      "center": [
        16.3665,
        48.2096
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3665,
          48.2096
        ]
      },
      "properties": {}
    }
  …
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Reverse Geocoding (nächste Adresse, Straße, Ort) {#reverse-geocoding-nearest-address-street-place}

`GET /v1/reverse`

Nächste Objekte um einen Punkt, das nächste zuerst. Mit `types` einschränken (z. B. `address`).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `lng` | query | string | ja | – | Länge (auch `lon`, `longitude`) |
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
    "herrengasse",
    "13",
    "wien"
  ],
  "features": [
    {
      "type": "Feature",
      "id": "address-1234567",
      "place_type": [
        "address"
      ],
      "place_name": "Herrengasse 13, 1010 Wien",
      "text": "Herrengasse 13",
      "center": [
        16.3665,
        48.2096
      ],
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3665,
          48.2096
        ]
      },
      "properties": {}
    }
  …
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Batch-Geocoding (bis zu 1000 Anfragen) {#batch-geocoding-up-to-1000-requests}

`POST /v1/geocode/batch`

JSON-Array; jeder Eintrag hat `q` oder strukturiert `street`/`housenumber`/`postcode`/`place`, optional `types` (String oder Liste), `limit` (1 bis 10), `proximity` [lon,lat], `bbox`. Antwort: Array von FeatureCollections in Eingabereihenfolge. Die Anfragen laufen gebündelt über die Multi-Search von Meilisearch (höchstens 4 parallel).

Höchstens 512 KB.

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

**Beispielantwort**

```json
[
  {
    "type": "FeatureCollection",
    "query": [
      "herrengasse",
      "13",
      "wien"
    ],
    "features": [
      {
        "type": "Feature",
        "id": "address-1234567",
        "place_type": [
          "address"
        ],
        "place_name": "Herrengasse 13, 1010 Wien",
        "text": "Herrengasse 13",
        "center": [
          16.3665,
          48.2096
        ],
        "geometry": {
          "type": "Point",
          "coordinates": [
            16.3665,
            48.2096
          ]
        },
        "properties": {}
  …
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `405` | Methode nicht erlaubt |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

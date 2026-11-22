# Adress-Autofill und Adressprüfung API

Source: https://docs.mapx.at/de/api/address/

> Adress-Autofill für Formulare und Adressvalidierung gegen das amtliche österreichische Adressregister: Endpunkte, Parameter und Beispiele.

Autofill vervollständigt eine Postadresse beim Tippen. Die Validierung prüft eine Adresse gegen das amtliche Register.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Adress-Autofill: Vorschläge {#address-autofill-suggestions}

`GET /v1/autofill/suggest`

Nur das amtliche Adressregister (BEV). Vervollständigt Straße, Hausnummer, Postleitzahl und Ort. FeatureCollection von Adressen im selben Format wie geocode (Schema `Place`, die Felder getrennt in `properties.address`, mit `distance` bei `proximity`). Ein vollständiger Ort oder eine PLZ nach der Hausnummer hat Vorrang wie bei geocode (`Hauptplatz 12a Graz` schlägt Adressen in Graz vor, die nächsten Nummern zuerst, wenn es 12a dort nicht gibt).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string | ja | – | Unvollständige Eingabe, z. B. `Hauptpl 3 Mel` |
| `session_token` | query | string |  | – | Beliebige Sitzungs-Id (bis 128 druckbare Zeichen), ohne Wirkung |
| `proximity` | query | string |  | – | lng,lat, bevorzugt nahe Adressen, liefert auch die Entfernung |
| `limit` | query | string |  | `5` | 1 bis 10, Standard 5 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "address-6444189",
      "geometry": {
        "type": "Point",
        "coordinates": [
          15.329874,
          48.227329
        ]
      },
      "properties": {
        "type": "address",
        "name": "Hauptplatz 3",
        "place_formatted": "3390 Melk, NÖ",
        "relevance": 0.97,
        "accuracy": "rooftop",
        "address": {
          "street": "Hauptplatz",
          "housenumber": "3",
          "postcode": "3390",
          "place": "Melk",
          "locality": "Melk",
          "municipality": "Melk",
          "district": "Melk",
          "state": "Niederösterreich"
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
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Adress-Autofill: gewählte Adresse abrufen {#address-autofill-retrieve-the-chosen-address}

`GET /v1/autofill/retrieve/{id}`

`id` eines Vorschlags (`address-<AdrCD>`). FeatureCollection mit dieser einen Adresse, im selben Format wie suggest (Schema `Place`). 404 bei unbekannter Id.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Adress-Id aus suggest |
| `session_token` | query | string |  | – | Wie bei suggest |

**Beispiel**

**curl**

```bash
ID=$(curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=1" | jq -r '.features[0].id')
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/autofill/retrieve/$ID" | jq
```

**JavaScript**

```js
const headers = { Authorization: "Bearer YOUR_TOKEN" };
const s = await (await fetch("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=1", { headers })).json();
const res = await fetch("https://api.mapx.at/v1/autofill/retrieve/" + encodeURIComponent(s.features[0].id), { headers });
const data = await res.json();
```

**Python**

```python
import requests

h = {"Authorization": "Bearer YOUR_TOKEN"}
s = requests.get("https://api.mapx.at/v1/autofill/suggest?q=Herrengasse%2013%20Wien&limit=1", headers=h).json()
res = requests.get("https://api.mapx.at/v1/autofill/retrieve/" + s["features"][0]["id"], headers=h)
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "address-6444189",
      "geometry": {
        "type": "Point",
        "coordinates": [
          15.329874,
          48.227329
        ]
      },
      "properties": {
        "type": "address",
        "name": "Hauptplatz 3",
        "place_formatted": "3390 Melk, NÖ",
        "relevance": 1,
        "accuracy": "rooftop",
        "address": {
          "street": "Hauptplatz",
          "housenumber": "3",
          "postcode": "3390",
          "place": "Melk",
          "locality": "Melk",
          "municipality": "Melk",
          "district": "Melk",
          "state": "Niederösterreich"
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
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe erforderlich. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Adressprüfung (Freitext oder strukturiert) {#address-validation-free-text-or-structured}

`GET /v1/address/validate`

Prüft eine Adresse gegen das amtliche Register. `status`: match | corrected | ambiguous | not_found. Liefert die erkannte Eingabe `input`, die normalisierte Adresse `address`, `confidence` (0 bis 1, bei `not_found` 0), `deviations[]` je Feld (kind: spelling | wrong | added | unknown), `coordinates`, `id` und `address_code`.

Versteht Str./Straße, Abkürzungen, Umlaute und ß; ein Hausnummernbereich (`1-3`) ist nicht die Nummer 13. Ein angegebener Ort oder eine angegebene PLZ hat Vorrang: Kandidaten anderswo zählen nur, wenn dort keiner liegt. Eine andere Hausnummer ist nie eine Korrektur: gibt es die Nummer in der Straße nicht, ist `status` `not_found`, `deviations` enthält `{"field": "housenumber", "kind": "unknown"}` und `candidates[]` listet die nächsten Nummern dieser Straße (bis zu 5), z. B. `street=Rathausplatz&housenumber=999&postcode=3100`. Bei `ambiguous` sind `address`, `coordinates`, `id` und `address_code` `null`, und `candidates[]` (bis zu 5, jeweils aufgebaut wie `address`) listet die möglichen Adressen, z. B. `q=Hauptplatz 3` (ohne Ort): `{"status": "ambiguous", "address": null, "id": null, "candidates": [{"id": "address-5111737", "full_address": "Hauptplatz 3, 6511 Zams", ...}, {"id": "address-6067644", "full_address": "Hauptplatz 3, 9170 Ferlach", ...}]}`. Bei `match` und `corrected` ist `candidates` leer.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string |  | – | Freitext, z. B. `Haubtplatz 1, 3390 Melk` (oder strukturierte Felder) |
| `street` | query | string |  | – | Straße |
| `housenumber` | query | string |  | – | Hausnummer |
| `postcode` | query | string |  | – | Postleitzahl |
| `place` | query | string |  | – | Ort |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/address/validate?q=Haubtplatz%201,%203390%20Melk", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "status": "corrected",
  "confidence": 0.96,
  "input": {
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  },
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)",
  "deviations": [
    {
      "field": "street",
      "input": "Haubtplatz",
      "expected": "Hauptplatz",
      "kind": "spelling"
    }
  ],
  "address": {
    "id": "address-6444187",
    "address_line1": "Hauptplatz 1",
    "street": "Hauptplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk",
    "municipality": "Melk",
    "state": "Niederösterreich",
    "full_address": "Hauptplatz 1, 3390 Melk",
    "coordinates": [
      15.329515,
      48.227317
    ]
  },
  "coordinates": [
    15.329515,
    48.227317
  ],
  "id": "address-6444187",
  "address_code": "6444187",
  "candidates": []
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `status` | string | `match`: wie angegeben gefunden; `corrected`: nach Korrekturen gefunden (siehe `deviations`); `ambiguous`: mehrere Adressen passen (siehe `candidates`); `not_found`: keine Adresse passt (ist nur die Hausnummer unbekannt, listet `candidates` die nächsten Nummern der Straße). |
| `confidence` | number | 0 bis 1; bei `not_found` 0. |
| `input` | object | Die Eingabe, wie sie verstanden wurde: `street`, `housenumber`, `postcode`, `place` (aus `q` zerlegt oder aus den strukturierten Feldern übernommen). |
| `address` | object,null | Die normalisierte Adresse (`id`, `address_line1`, `street`, `housenumber`, `postcode`, `place`, `municipality`, `state`, `full_address`, `coordinates`); `null`, außer bei `match` oder `corrected`. |
| `deviations` | array | Je korrigiertem Feld: `field`, `input`, `expected`, `kind` (spelling, wrong oder added); bei `not_found` wegen unbekannter Hausnummer `{field: housenumber, kind: unknown}`. |
| `coordinates` | array,null | [lng, lat] der Adresse, `null`, außer bei `match` oder `corrected`. |
| `id` | string,null | Adress-Id `address-<AdrCD>` (wie bei Autofill und Geocoding). |
| `address_code` | string,null | Adresscode des BEV-Adressregisters. |
| `candidates` | array | Bei `ambiguous`: bis zu 5 mögliche Adressen; bei `not_found` wegen unbekannter Hausnummer: bis zu 5 nächste Nummern der Straße; jeweils aufgebaut wie `address`; sonst leer. |
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe erforderlich. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Adressprüfung (JSON-Body) {#address-validation-json-body}

`POST /v1/address/validate`

Wie GET, mit den Feldern in einem JSON-Objekt (`q` oder `street`, `housenumber`, `postcode`, `place`). Höchstens 16 KB.

Braucht einen geheimen Token (`mx_sk_`); öffentliche Tokens erhalten `403 pk-not-allowed`, im Playground daher nicht ausführbar.

**Request-Body (JSON)**

```json
{
  "street": "Haubtplatz",
  "housenumber": "1",
  "postcode": "3390",
  "place": "Melk"
}
```

**Beispiel**

**curl**

```bash
curl -s -X POST "https://api.mapx.at/v1/address/validate" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"street":"Haubtplatz","housenumber":"1","postcode":"3390","place":"Melk"}' | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/address/validate", {
  method: "POST",
  headers: { "Authorization": "Bearer YOUR_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify({
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  }),
});
const data = await res.json();
```

**Python**

```python
import requests

res = requests.post("https://api.mapx.at/v1/address/validate", headers={"Authorization": "Bearer YOUR_TOKEN"}, json={
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  })
data = res.json()
```

**Beispielantwort**

```json
{
  "status": "corrected",
  "confidence": 0.96,
  "input": {
    "street": "Haubtplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk"
  },
  "attribution": "mapx.at; BEV Address Register, Geographical Names, Administrative Boundaries, DKM (CC BY 4.0)",
  "deviations": [
    {
      "field": "street",
      "input": "Haubtplatz",
      "expected": "Hauptplatz",
      "kind": "spelling"
    }
  ],
  "address": {
    "id": "address-6444187",
    "address_line1": "Hauptplatz 1",
    "street": "Hauptplatz",
    "housenumber": "1",
    "postcode": "3390",
    "place": "Melk",
    "municipality": "Melk",
    "state": "Niederösterreich",
    "full_address": "Hauptplatz 1, 3390 Melk",
    "coordinates": [
      15.329515,
      48.227317
    ]
  },
  "coordinates": [
    15.329515,
    48.227317
  ],
  "id": "address-6444187",
  "address_code": "6444187",
  "candidates": []
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `status` | string | `match`: wie angegeben gefunden; `corrected`: nach Korrekturen gefunden (siehe `deviations`); `ambiguous`: mehrere Adressen passen (siehe `candidates`); `not_found`: keine Adresse passt (ist nur die Hausnummer unbekannt, listet `candidates` die nächsten Nummern der Straße). |
| `confidence` | number | 0 bis 1; bei `not_found` 0. |
| `input` | object | Die Eingabe, wie sie verstanden wurde: `street`, `housenumber`, `postcode`, `place` (aus `q` zerlegt oder aus den strukturierten Feldern übernommen). |
| `address` | object,null | Die normalisierte Adresse (`id`, `address_line1`, `street`, `housenumber`, `postcode`, `place`, `municipality`, `state`, `full_address`, `coordinates`); `null`, außer bei `match` oder `corrected`. |
| `deviations` | array | Je korrigiertem Feld: `field`, `input`, `expected`, `kind` (spelling, wrong oder added); bei `not_found` wegen unbekannter Hausnummer `{field: housenumber, kind: unknown}`. |
| `coordinates` | array,null | [lng, lat] der Adresse, `null`, außer bei `match` oder `corrected`. |
| `id` | string,null | Adress-Id `address-<AdrCD>` (wie bei Autofill und Geocoding). |
| `address_code` | string,null | Adresscode des BEV-Adressregisters. |
| `candidates` | array | Bei `ambiguous`: bis zu 5 mögliche Adressen; bei `not_found` wegen unbekannter Hausnummer: bis zu 5 nächste Nummern der Straße; jeweils aufgebaut wie `address`; sonst leer. |
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe erforderlich. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `502` | Vorgelagerter Dienst nicht erreichbar |

## Playground

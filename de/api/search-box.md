# Search Box API: Suche beim Tippen

Source: https://docs.mapx.at/de/api/search-box/

> Search-Box-API für Autovervollständigung: Vorschläge beim Tippen, Details eines Treffers und Orte nach Kategorie, mit Parametern und Beispielen.

Suche mit Vorschlägen für Adressen und Orte. `suggest` beim Tippen aufrufen, danach `retrieve` für den gewählten Treffer.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Search Box: Vorschläge {#search-box-suggestions}

`GET /v1/searchbox/suggest`

Suche beim Tippen über Adressen, Straßen, Orte und amtliche POIs. FeatureCollection im selben Format wie geocode (Schema `Place`, mit Punktgeometrie); POIs tragen zusätzlich `category` und `maki`, Haltestellen `stop_id` und `modes`. `/retrieve/{id}` liefert einen einzelnen Treffer über seine `id`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string | ja | – | Suchtext |
| `session_token` | query | string |  | – | Beliebige Sitzungs-Id (bis 128 Zeichen), ohne Wirkung |
| `proximity` | query | string |  | – | lon,lat; gleiche Reihung wie bei geocode (samt den nächsten Orten einer in `q` genannten Kategorie), liefert auch die Entfernung |
| `types` | query | string |  | – | region, postcode, district, place, locality, street, address, poi, road_ref (country, neighborhood, category haben keine Wirkung) |
| `poi_category` | query | string |  | – | Kategorie-Ids, durch Komma getrennt (z. B. `gas_station`), siehe `/v1/searchbox/list/category` |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lon -180 bis 180, lat -90 bis 90; sonst 400 `invalid-parameter`) |
| `limit` | query | string |  | `5` | 1 bis 10, Standard 5 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&proximity=16.3725,48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "place-7158611",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3731,
          48.2085
        ]
      },
      "properties": {
        "type": "place",
        "name": "Stephansdom",
        "place_formatted": "Kirche und Kapelle, Wien",
        "relevance": 0.99,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "zoom": 16,
        "class": "kirche",
        "category": "place_of_worship_church",
        "maki": "religious-christian"
      }
    }
  ],
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `query` | array | Nur Geocoding: die Suchwörter, bzw. [lon, lat] bei Rückwärtssuche und Koordinaten als Suchtext. |
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

## Search Box: Vorschlag auflösen {#search-box-resolve-a-suggestion}

`GET /v1/searchbox/retrieve/{id}`

FeatureCollection mit diesem einen Treffer, im selben Format wie suggest und geocode (Schema `Place`); 404 bei unbekannter Id.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | `id` eines Treffers aus suggest, geocode oder category |
| `session_token` | query | string |  | – | Wie bei suggest |
| `proximity` | query | string |  | – | lon,lat, nur für die Entfernung |

**Beispiel**

**curl**

```bash
ID=$(curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&limit=1" | jq -r '.features[0].id')
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/retrieve/$ID" | jq
```

**JavaScript**

```js
const headers = { Authorization: "Bearer YOUR_TOKEN" };
const s = await (await fetch("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&limit=1", { headers })).json();
const res = await fetch("https://api.mapx.at/v1/searchbox/retrieve/" + encodeURIComponent(s.features[0].id), { headers });
const data = await res.json();
```

**Python**

```python
import requests

h = {"Authorization": "Bearer YOUR_TOKEN"}
s = requests.get("https://api.mapx.at/v1/searchbox/suggest?q=Stephansdom&limit=1", headers=h).json()
res = requests.get("https://api.mapx.at/v1/searchbox/retrieve/" + s["features"][0]["id"], headers=h)
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "place-7158611",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3731,
          48.2085
        ]
      },
      "properties": {
        "type": "place",
        "name": "Stephansdom",
        "place_formatted": "Kirche und Kapelle, Wien",
        "relevance": 0.99,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "zoom": 16,
        "class": "kirche",
        "category": "place_of_worship_church",
        "maki": "religious-christian"
      }
    }
  ],
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `query` | array | Nur Geocoding: die Suchwörter, bzw. [lon, lat] bei Rückwärtssuche und Koordinaten als Suchtext. |
| `features` | array | Treffer, der beste bzw. nächste zuerst. Jeder ist ein GeoJSON-Punkt-Feature mit stabiler `id` (und `bbox` bei Flächen); `properties` enthalten immer `type`, `name`, `place_formatted`, `relevance`, `accuracy` und `address`, je nach Treffer auch `distance`, `zoom`, `class`, `category`, `maki`, `stop_id`, `modes`, `road_ref` (Schema `Place`). |
| `warning` | string | Nur Rückwärtssuche: der Punkt liegt außerhalb Österreichs. |
| `attribution` | string | Datenquellen (CC BY 4.0), Quellenangabe verpflichtend. |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Search Box: Orte einer Kategorie {#search-box-places-of-a-category}

`GET /v1/searchbox/category/{category}`

Orte der Kategorie (mehrere durch Komma getrennt), mit `proximity` die nächsten zuerst (dann mit `properties.distance`). Ids wie in `/v1/searchbox/list/category` (z. B. `gas_station`, `train_station`). Gleiches Format wie suggest (Schema `Place`).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `category` | path | string | ja | – | Kategorie-Id(s) |
| `proximity` | query | string |  | – | lon,lat |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat (minlon &lt; maxlon, minlat &lt; maxlat, lon -180 bis 180, lat -90 bis 90; sonst 400 `invalid-parameter`) |
| `limit` | query | string |  | `10` | 1 bis 25, Standard 10 |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/category/gas_station?proximity=16.3725,48.2085&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "id": "poi-51234",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.37,
          48.21
        ]
      },
      "properties": {
        "type": "poi",
        "name": "Tankstelle Am Stadtrand",
        "place_formatted": "Tankstelle, Wien",
        "relevance": 1,
        "accuracy": null,
        "address": {
          "municipality": "Wien",
          "district": "Wien (Stadt)",
          "state": "Wien"
        },
        "distance": 412,
        "zoom": 16,
        "category": "gas_station",
        "maki": "fuel"
      }
    }
  ],
  "attribution": "mapx.at; Daten: BEV – Adressregister, Geonamen, Verwaltungsgrenzen, DKM (CC BY 4.0)"
}
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `query` | array | Nur Geocoding: die Suchwörter, bzw. [lon, lat] bei Rückwärtssuche und Koordinaten als Suchtext. |
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

## Search Box: Kategorienliste {#search-box-category-list}

`GET /v1/searchbox/list/category`

Stabile Kategorie-Ids (`canonical_id`), Name und Maki-Icon.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `language` | query | string |  | – | de (Standard) oder en |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/list/category?language=en" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/list/category?language=en", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/list/category?language=en", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "listItems": [
    {
      "canonical_id": "gas_station",
      "icon": "fuel",
      "name": "Tankstelle"
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Playground

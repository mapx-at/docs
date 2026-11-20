# Search Box API: Suche beim Tippen

Source: https://docs.mapx.at/de/api/search-box/

> Search-Box-API für Autovervollständigung: Vorschläge beim Tippen, Details eines Treffers und Orte nach Kategorie, mit Parametern und Beispielen.

Suche mit Vorschlägen für Adressen und Orte. `suggest` beim Tippen aufrufen, danach `retrieve` für den gewählten Treffer.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Suche](https://docs.mapx.at/de/guides/search.md)

## Search Box: Vorschläge {#search-box-suggestions}

`GET /v1/searchbox/suggest`

Wie geocode, aber eine Vorschlagsliste ohne Koordinaten (`mapbox_id` für `/retrieve`), mit Kategorie (`poi_category`, `poi_category_ids`, `maki`) und `context`. Amtliche POIs, keine OSM-Daten.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `q` | query | string | ja | – | Suchtext |
| `session_token` | query | string |  | – | Beliebige Sitzungs-Id (bis 128 Zeichen), ohne Wirkung |
| `proximity` | query | string |  | – | lon,lat; gleiche Reihung wie bei geocode, liefert auch die Entfernung |
| `types` | query | string |  | – | region, postcode, district, place, locality, street, address, poi, road_ref (country, neighborhood, category haben keine Wirkung) |
| `poi_category` | query | string |  | – | Kategorie-Ids, durch Komma getrennt (z. B. `gas_station`), siehe `/v1/searchbox/list/category` |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
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
  "suggestions": [
    {
      "name": "Stephansdom",
      "mapbox_id": "poi-4711",
      "feature_type": "poi",
      "place_formatted": "Place of worship, Wien",
      "poi_category": [
        "place_of_worship"
      ]
    }
  ],
  "attribution": "..."
}
```

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

FeatureCollection mit einem Objekt (Punkt, bbox, Koordinaten, Kontext); 404 bei unbekannter Id.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | `mapbox_id` aus suggest |
| `session_token` | query | string |  | – | Wie bei suggest |
| `proximity` | query | string |  | – | lon,lat, nur für die Entfernung |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/searchbox/retrieve/place-6456531" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/searchbox/retrieve/place-6456531", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/searchbox/retrieve/place-6456531", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.3731,
          48.2085
        ]
      },
      "properties": {
        "name": "Stephansdom"
      }
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
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Search Box: Orte einer Kategorie {#search-box-places-of-a-category}

`GET /v1/searchbox/category/{category}`

Orte der Kategorie (mehrere durch Komma getrennt), mit `proximity` die nächsten zuerst. Ids wie in `/v1/searchbox/list/category` (z. B. `gas_station`, `train_station`).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `category` | path | string | ja | – | Kategorie-Id(s) |
| `proximity` | query | string |  | – | lon,lat |
| `bbox` | query | string |  | – | minlon,minlat,maxlon,maxlat |
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
      "geometry": {
        "type": "Point",
        "coordinates": [
          16.37,
          48.21
        ]
      },
      "properties": {
        "name": "Tankstelle Am Stadtrand"
      }
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

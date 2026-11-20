# Tilequery API

Source: https://docs.mapx.at/de/api/tilequery/

> Tilequery-API: Objekte um einen Punkt aus beliebigen Vektor-Tilesets, mit Parametern und Beispielen.

Tilequery liefert die Vektorobjekte nahe einer Koordinate.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Karten](https://docs.mapx.at/de/guides/maps.md)

## Tilequery: Vektorobjekte um einen Punkt {#tilequery-vector-features-around-a-point}

`GET /v1/tilequery/{tilesets}/{coordinate}.json`

Liest die Vektorkachel in der höchsten Zoomstufe des Tilesets (und für einen Radius die Nachbarkacheln). FeatureCollection, das nächste Objekt zuerst, `properties.tilequery = {distance (m), geometry (point|linestring|polygon), layer}`. Polygone, die den Punkt enthalten, haben die Distanz 0.

Mehrere Tilesets, durch Komma getrennt, ergänzen `tilequery.tileset`; `dtm`/`dsm` liefern die Höhe als Punktobjekt. Geometrien sind auf die Kachel zugeschnitten.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tilesets` | path | string | ja | – | Tileset-Namen, durch Komma getrennt (höchstens 15), z. B. `streets,contours` |
| `coordinate` | path | string | ja | – | lon,lat (der Pfad endet auf `.json`) |
| `radius` | query | string |  | `0` | Meter, 0 bis 2000, Standard 0 |
| `limit` | query | string |  | `5` | 1 bis 50, Standard 5 |
| `layers` | query | string |  | – | Layer-Namen, durch Komma getrennt |
| `geometry` | query | string |  | – | point, linestring oder polygon |
| `dedupe` | query | string |  | – | true (Standard) entfernt Duplikate über Kachelgrenzen |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tilequery/streets/16.3725,48.2085.json?radius=50&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/tilequery/streets/16.3725,48.2085.json?radius=50&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/tilequery/streets/16.3725,48.2085.json?radius=50&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
          16.37253,
          48.20852
        ]
      },
      "properties": {
        "name": "Herrengasse",
        "tilequery": {
          "distance": 2.9,
          "geometry": "linestring",
          "layer": "road_l"
        }
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

## Playground

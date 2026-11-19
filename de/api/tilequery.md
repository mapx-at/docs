# Tilequery und Parcel API

Source: https://docs.mapx.at/de/api/tilequery/

> Tilequery-API: Objekte um einen Punkt aus beliebigen Vektor-Tilesets und das Katastergrundstück an einem Punkt (Parcel-API), mit Parametern und Beispielen.

Tilequery liefert die Vektorobjekte nahe einer Koordinate. Parcel liefert das Grundstück (Fläche, Nummer, Katastralgemeinde) an einem Punkt.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Grundstücke](https://docs.mapx.at/de/guides/parcels.md)

## Tilequery: Vektorobjekte um einen Punkt {#tilequery-vector-features-around-a-point}

`GET /v1/tilequery/{tilesets}/{coordinate}.json`

Liest die Vektorkachel in der höchsten Zoomstufe des Tilesets (und für einen Radius die Nachbarkacheln). FeatureCollection, das nächste Objekt zuerst, `properties.tilequery = {distance (m), geometry (point|linestring|polygon), layer}`. Polygone, die den Punkt enthalten, haben die Distanz 0.

Mehrere Tilesets, durch Komma getrennt, ergänzen `tilequery.tileset`; `dtm`/`dsm` liefern die Höhe als Punktobjekt. Geometrien sind auf die Kachel zugeschnitten.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tilesets` | path | string | ja | – | Tileset-Namen, durch Komma getrennt (höchstens 15), z. B. `streets,cadastre` |
| `coordinate` | path | string | ja | – | lon,lat (der Pfad endet auf `.json`) |
| `radius` | query | string |  | `0` | Meter, 0 bis 2000, Standard 0 |
| `limit` | query | string |  | `5` | 1 bis 50, Standard 5 |
| `layers` | query | string |  | – | Layer-Namen, durch Komma getrennt |
| `geometry` | query | string |  | – | point, linestring oder polygon |
| `dedupe` | query | string |  | – | true (Standard) entfernt Duplikate über Kachelgrenzen |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
        "gstnr": "817",
        "kgnr": 1004,
        "tilequery": {
          "distance": 2.9,
          "geometry": "linestring",
          "layer": "parcel_l"
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

## Grundstück an einem Punkt (BEV DKM) {#parcel-at-a-point-bev-dkm}

`GET /v1/parcel/{coordinate}`

Grundstück (`parcel_number`, `cadastral_municipality_id`, `area_m2`, `is_building_land`, `source`, dazu `parcel_id` und `links.dossier`) als Polygon, berechnet aus den Grenzlinien der Katasterkacheln (die Fläche weicht um Promille vom Wert im Register ab). Auf Gebäuden kommt das Bauflächengrundstück (`parcel_number` beginnt mit einem Punkt).

Grundstücke, die über den Kachelausschnitt (etwa 2 km) hinausreichen, fehlen. Außerhalb der Katasterdaten eine leere Liste. Mit `include=dossier` ist jedes Objekt das vollständige Dossier von `/v1/parcels/{id}` (Zonenkosten 4 plus Dossier).

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat oder lon,lat.json |
| `radius` | query | number (0–100) |  | `0` | Meter, 0 bis 100, Standard 0 (andere Werte: 400). Nur zur Kompatibilität angenommen; das Ergebnis ist immer das Grundstück, das den Punkt enthält |
| `include` | query | `dossier` |  | – | `dossier` liefert je Objekt das vollständige Dossier |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/parcel/15.3294,48.2272", headers={"Authorization": "Bearer YOUR_TOKEN"})
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
        "type": "Polygon",
        "coordinates": [
          [
            [
              16.3721,
              48.2081
            ],
            [
              16.3727,
              48.2081
            ],
            [
              16.3727,
              48.2086
            ],
            [
              16.3721,
              48.2081
            ]
          ]
        ]
      },
      "properties": {
  …
```

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Grundstücksdossier (BEV DKM und weitere offene Daten) {#parcel-dossier-bev-dkm-and-further-open-data}

`GET /v1/parcels/{id}`

Alles, was die offenen Daten über ein Grundstück sagen, in einem Aufruf: Verwaltungszuordnung, Fläche (`area_m2` in der Ebene des Gauß-Krüger-Systems, wie der Kataster sie berechnet, `area_ellipsoidal_m2`), Anteile der Nutzungen aus der DKM, Gebäude (Anzahl, Grundfläche, Höhen aus dem BEV-DLM, wo vorhanden), Geländestatistik (minimale, mittlere und maximale Höhe sowie mittlere und maximale Neigung aus dem Geländemodell, Raster von bis zu 250 Punkten im Grundstück), dem Grundstück zugeordnete Adressen, nächste Haltestelle des öffentlichen Verkehrs (`null`, wenn der Router nicht verfügbar ist), Jahr des Orthofoto-Bildflugs, `climate` (das Objekt von `/v1/climate/{coordinate}` am inneren Punkt des Grundstücks: Sonnenpotenzial, Klimanormale, Hitze- und Schneetage, Dürre; aus einem Cache, die Abschnitte sind `null` mit `status: partial`, bis die Daten eines neuen Gebiets geladen sind), die Geometrie als GeoJSON und ein fertiger `links.static_image`.

Die ID kommt aus `properties.parcel_id` von Grundstückstreffern der Suche und aus `/v1/parcel/{coordinate}`; die Form `14143-288/4` (Schrägstrich prozentcodiert) wird ebenfalls angenommen. 404 bei unbekannten Grundstücken, 503, solange der Dossier-Dienst nicht verfügbar ist. Stichtag des Katasters: siehe `properties.reference_date`.

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Grundstücks-ID `<KG-Nummer>-<Grundstücksnummer>`, `/` als `_` geschrieben, Bauflächen mit führendem Punkt: `14143-288_4`, `14143-.98` |

**Beispiel**

**curl**

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcels/14143-288_4" | jq
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/parcels/14143-288_4", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/parcels/14143-288_4", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

**Beispielantwort**

```json
{
  "type": "Feature",
  "id": "14143-288_4",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [
          15.3414,
          48.2191
        ],
        [
          15.3447,
          48.2191
        ],
        [
          15.3447,
          48.2227
        ],
        [
          15.3414,
          48.2191
        ]
      ]
    ]
  },
  "bbox": [
    15.3414,
    48.2191,
  …
```

| Feld | Typ | Beschreibung |
|---|---|---|
| `id` | string | Grundstücks-ID: `<KG-Nummer, 5 Ziffern>-<Grundstücksnummer>` mit `/` als `_`, Bauflächen beginnen mit einem Punkt, z. B. `14143-288_4`, `14143-.98` |

**Fehler** (Details: [Fehlerkatalog](https://docs.mapx.at/de/api/errors.md))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `401` | Token fehlt oder ungültig (`missing-token`, `invalid-token`, `token-expired`) |
| `403` | Token nicht erlaubt (`origin-not-allowed`, `origin-missing`, `pk-not-allowed`, `app-not-allowed`, `token-blocked`, `ip-not-allowed`) |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

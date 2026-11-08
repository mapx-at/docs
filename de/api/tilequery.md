# Tilequery und Parcel API

Objekte um einen Punkt aus beliebigen Vektor-Tilesets und das Grundstück an einem Punkt.

{/* Generiert von scripts/gen-reference.mjs aus openapi.json, nicht von Hand ändern. */}

Tilequery liefert die Vektorobjekte nahe einer Koordinate. Parcel liefert das Grundstück (Fläche, Nummer, Katastralgemeinde) an einem Punkt.

Basis-URL: `https://api.mapx.at` · Rate-Limit-Zone: `search` (10/s) · Guide: [Grundstücke](/de/guides/parcels/)

## Tilequery: vector features around a point

`GET /v1/tilequery/{tilesets}/{coordinate}.json`

Reads the vector tile at the tileset's highest zoom (and neighbouring tiles for a radius). FeatureCollection, nearest feature first, `properties.tilequery = {distance (m), geometry (point|linestring|polygon), layer}`. Polygons containing the point have distance 0.

<details>
<summary>Mehr</summary>

Several comma-separated tilesets add `tilequery.tileset`; `dtm`/`dsm` return the height as a point feature. Geometries are clipped to the tile.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `tilesets` | path | string | ja | – | Tileset names, comma-separated (at most 15), e.g. `streets,cadastre` |
| `coordinate` | path | string | ja | – | lon,lat (the path ends in `.json`) |
| `radius` | query | string |  | `0` | Meters, 0 to 2000, default 0 |
| `limit` | query | string |  | `5` | 1 to 50, default 5 |
| `layers` | query | string |  | – | Comma-separated layer names |
| `geometry` | query | string |  | – | point, linestring or polygon |
| `dedupe` | query | string |  | – | true (default) removes duplicates across tile borders |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

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
        "name": "Herrengasse",
        "tilequery": {
          "distance": 3.2,
          "geometry": "linestring",
          "layer": "road_l"
        }
      }
    }
  ]
}
```

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |

## Parcel at a point (BEV DKM)

`GET /v1/parcel/{coordinate}`

Parcel (`parcel_number`, `cadastral_municipality_id`, `area_m2`, `is_building_land`, `source`, plus `parcel_id` and `links.dossier`; the old fields `gstnr`, `kgnr`, `bauflaeche`, `quelle` are deprecated and still present until 2027-05-31) as polygon, computed from the boundary lines of the cadastre tiles (area deviates by per mille from the register value).

<details>
<summary>Mehr</summary>

Buildings return the building parcel (`parcel_number` starts with a dot). Parcels extending beyond the tile window (about 2 km) are missing. Empty list outside the cadastre data. With `include=dossier` each feature is the full dossier of `/v1/parcels/{id}` (zone cost 4 plus the dossier).

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat or lon,lat.json |
| `radius` | query | string |  | `0` | Meters, 0 to 100, default 0 |
| `include` | query | `dossier` |  | – | `dossier` returns the full dossier per feature |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/parcel/15.3294,48.2272", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

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

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Parcel dossier (BEV DKM and further open data)

`GET /v1/parcels/{id}`

Everything the open data say about one parcel in one call: administrative assignment, area (`area_m2` in the plane of the Gauss-Krueger system as the cadastre computes it, `area_ellipsoidal_m2`), land use shares from the DKM, buildings (count, footprint, heights from BEV DLM where available), terrain statistics (min, mean, max elevation and mean/max slope from the terrain model, grid of up to 250 points inside the parcel), addresses assigned to the parcel, nearest public transit stop (`null` if the router is unavailable), year of the orthophoto flight, `climate` (the object of `/v1/climate/{coordinate}` at the inner point of the parcel: solar potential, normals, hot and snow days, drought; taken from a cache, sections are `null` with `status: partial` until the data of a new area are loaded), the geometry as GeoJSON and a ready-made `links.static_image`.

<details>
<summary>Mehr</summary>

The ID comes from `properties.parcel_id` of parcel search results and `/v1/parcel/{coordinate}`; the form `14143-288/4` (percent-encoded slash) is accepted as well. 404 for unknown parcels, 503 while the dossier service is unavailable. Cadastre key date: see `properties.reference_date`.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `id` | path | string | ja | – | Parcel ID `<KG number>-<parcel number>`, `/` written as `_`, building parcels with a leading dot: `14143-288_4`, `14143-.98` |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" "https://api.mapx.at/v1/parcels/14143-288_4" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/parcels/14143-288_4", { headers: { Authorization: "Bearer YOUR_TOKEN" } });
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/parcels/14143-288_4", headers={"Authorization": "Bearer YOUR_TOKEN"})
data = res.json()
```

</TabItem>
</Tabs>

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
| `id` | string | Parcel ID: `<KG number, 5 digits>-<parcel number>` with `/` written as `_`, building parcels start with a dot, e.g. `14143-288_4`, `14143-.98` |

**Fehler** (Details: [Fehlerkatalog](/de/api/errors/))

| Status | Bedeutung |
|---|---|
| `400` | Parameter fehlt oder ist ungültig |
| `404` | Nicht gefunden (Name, Pfad oder Position) |
| `429` | Rate-Limit überschritten, `Retry-After` abwarten |
| `503` | Daten vorübergehend nicht verfügbar |

## Playground

<Playground group="tilequery" lang="de" />
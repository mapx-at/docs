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
curl -s "https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/tilequery/cadastre/16.3725,48.2085.json?radius=50&limit=3")
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

Parcel (`parcel_number`, `cadastral_municipality_id`, `area_m2`, `is_building_land`, `source`; the old fields `gstnr`, `kgnr`, `bauflaeche`, `quelle` are deprecated and still present until 2027-05-31) as polygon, computed from the boundary lines of the cadastre tiles (area deviates by per mille from the register value).

<details>
<summary>Mehr</summary>

Buildings return the building parcel (`parcel_number` starts with a dot). Parcels extending beyond the tile window (about 2 km) are missing. Empty list outside the cadastre data.

</details>

**Parameter**

| Name | Ort | Typ | Pflicht | Standard | Beschreibung |
|---|---|---|---|---|---|
| `coordinate` | path | string | ja | – | lon,lat or lon,lat.json |

**Beispiel**

<Tabs syncKey="lang">
<TabItem label="curl">

```bash
curl -s "https://api.mapx.at/v1/parcel/15.3294,48.2272" | jq
```

</TabItem>
<TabItem label="JavaScript">

```js
const res = await fetch("https://api.mapx.at/v1/parcel/15.3294,48.2272");
const data = await res.json();
```

</TabItem>
<TabItem label="Python">

```python

res = requests.get("https://api.mapx.at/v1/parcel/15.3294,48.2272")
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

## Playground

<Playground group="tilequery" lang="de" />
# Style specification


## Style IDs

| ID | Look |
|---|---|
| `streets`, `streets-dawn`, `streets-dusk`, `streets-night` | Base map in four light moods |
| `satellite`, `satellite-streets` | Orthophoto, optionally with roads and labels |
| `outdoors` | Contours, hillshade, trails |
| `traffic-day` | Base map with the traffic layer |

Styles are MapLibre style documents (version 8) at `https://api.mapx.at/v1/styles/{id}`. Their metadata keys are `mapx:style` and `mapx:api_style`. See the [Styles API](/api/styles/).

## Layers of `streets`

Every layer of [`streets`](/tilesets/streets/) with its fields. For fields with a closed list, all values are listed. Layer names end in `_a` (polygon), `_l` (line) or `_p` (point).

Common fields: `min_zoom` is the first zoom level at which a feature is intended to be shown (use it in filters for an even density); `sort_rank` orders features within a layer (smaller = more important or drawn first).

## Land use and buildings

### landuse_a



### building_a



### ski_a



## Transport

### road_l



### rail_l



### aerialway_l



### transit_p



### airport_p



## Water

### water_a



### water_l



### water_label_p



### water_label_l



## Boundaries

### boundary_l



### boundary_label_p



## Places and names

### place_p



### natural_p



### poi_p



### housenumber_p




# Style-Spezifikation

Layer, Felder und Werte des Tilesets streets sowie die fertigen Style-IDs.

## Style-IDs

| ID | Aussehen |
|---|---|
| `streets`, `streets-dawn`, `streets-dusk`, `streets-night` | Basiskarte in vier Lichtstimmungen |
| `satellite`, `satellite-streets` | Orthophoto, wahlweise mit Straßen und Beschriftung |
| `outdoors` | Höhenlinien, Schummerung, Wege |
| `traffic-day` | Basiskarte mit Verkehrsebene |

Styles sind MapLibre-Style-Dokumente (Version 8) unter `https://api.mapx.at/v1/styles/{id}`. Die Metadaten heißen `mapx:style` und `mapx:api_style`. Siehe [Styles API](/de/api/styles/).

## Layer von `streets`

Jeder Layer der [`streets`](/de/tilesets/streets/) mit seinen Feldern. Bei Feldern mit geschlossener Liste stehen alle möglichen Werte dabei. Layernamen enden auf `_a` (Fläche), `_l` (Linie) oder `_p` (Punkt).

Gemeinsame Felder: `min_zoom` ist die erste Zoomstufe, ab der ein Objekt gezeigt werden soll (für Filter mit gleichmäßiger Dichte); `sort_rank` ordnet Objekte innerhalb eines Layers (kleiner = wichtiger bzw. zuerst gezeichnet).

## Fläche und Nutzung

### landuse_a

<FeldReferenz ts="basis" layer="landuse_a" />

### building_a

<FeldReferenz ts="basis" layer="building_a" />

### ski_a

<FeldReferenz ts="basis" layer="ski_a" />

## Verkehr

### road_l

<FeldReferenz ts="basis" layer="road_l" />

### rail_l

<FeldReferenz ts="basis" layer="rail_l" />

### aerialway_l

<FeldReferenz ts="basis" layer="aerialway_l" />

### transit_p

<FeldReferenz ts="basis" layer="transit_p" />

### airport_p

<FeldReferenz ts="basis" layer="airport_p" />

## Gewässer

### water_a

<FeldReferenz ts="basis" layer="water_a" />

### water_l

<FeldReferenz ts="basis" layer="water_l" />

### water_label_p

<FeldReferenz ts="basis" layer="water_label_p" />

### water_label_l

<FeldReferenz ts="basis" layer="water_label_l" />

## Grenzen

### boundary_l

<FeldReferenz ts="basis" layer="boundary_l" />

### boundary_label_p

<FeldReferenz ts="basis" layer="boundary_label_p" />

## Namen und Orte

### place_p

<FeldReferenz ts="basis" layer="place_p" />

### natural_p

<FeldReferenz ts="basis" layer="natural_p" />

### poi_p

<FeldReferenz ts="basis" layer="poi_p" />

### housenumber_p

<FeldReferenz ts="basis" layer="housenumber_p" />
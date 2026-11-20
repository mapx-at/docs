# Rest areas tileset (ASFINAG)

Source: https://docs.mapx.at/tilesets/rest-areas/

> Rest areas and truck parking of ASFINAG as vector tiles (tileset rest-areas) and as the search category rastplatz in the mapx search.

Rest, commuter and truck parking sites along motorways and expressways from the ASFINAG traffic data, 314 locations.

> **Warning:** **Attribution**
> 
> Data source: ASFINAG, rest, commuter and truck parking, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Changes: selection and reshaping of fields.

| | |
|---|---|
| Tileset | `rest-areas`, vector, z7 to z14 |
| Layer | `rest_area_p` (points) with `id`, `name`, `road`, `car` and `lorry` (spaces), `free`, `usage`, `min_zoom` |
| Style | group `rest_areas`, off by default: point, name from z12; sites with truck spaces darker |
| Search | category `rastplatz` in [Search Box](https://docs.mapx.at/api/search-box.md) and geocoding |
| Query | [Tilequery](https://docs.mapx.at/api/tilequery.md) with `rest-areas` as tileset |

Real-time occupancy is not part of the tiles (yet).

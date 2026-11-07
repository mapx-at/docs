# Rest areas

Rest areas and truck parking of ASFINAG as vector tiles (tileset rastanlagen) and search category rastplatz.

Rest, commuter and truck parking sites along motorways and expressways from the ASFINAG traffic data, 314 locations.

> Data source: ASFINAG, rest, commuter and truck parking, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Changes: selection and reshaping of fields.

| | |
|---|---|
| Tileset | `rastanlagen`, vector, z7 to z14 |
| Layer | `rastanlagen` (points) with `id`, `name`, `road`, `car` and `lorry` (spaces), `free`, `usage`, `min_zoom` |
| Style | group `rest_areas`, off by default: point, name from z12; sites with truck spaces darker |
| Search | category `rastplatz` in [Search Box](/en/api/searchbox/) and geocoding |
| Query | [Tilequery](/en/api/tilequery/) with `rastanlagen` as tileset |

Real-time occupancy is not part of the tiles (yet).
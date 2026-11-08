# Rest areas


Rest, commuter and truck parking sites along motorways and expressways from the ASFINAG traffic data, 314 locations.

> **Attribution**

Data source: ASFINAG, rest, commuter and truck parking, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Changes: selection and reshaping of fields.


| | |
|---|---|
| Tileset | `rastanlagen`, vector, z7 to z14 |
| Layer | `rastanlagen` (points) with `id`, `name`, `road`, `car` and `lorry` (spaces), `free`, `usage`, `min_zoom` |
| Style | group `rest_areas`, off by default: point, name from z12; sites with truck spaces darker |
| Search | category `rastplatz` in [Search Box](/api/search-box/) and geocoding |
| Query | [Tilequery](/api/tilequery/) with `rastanlagen` as tileset |

Real-time occupancy is not part of the tiles (yet).


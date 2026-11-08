# Sharing locations

Sharing stations (cargo bikes, bikes) of ÖBB as vector tiles (tileset sharing) and search category sharing.

Locations of sharing stations from the data set "Shared-Mobility Standorte 2026" of ÖBB-Personenverkehr AG. It only contains sites ÖBB operates and owns itself; currently seven stations in Korneuburg and Leoben (as of 2026-11-07).

> Data source: ÖBB-Personenverkehr AG, Shared-Mobility Standorte, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de).

| | |
|---|---|
| Tileset | `sharing`, vector, z8 to z14 |
| Layer | `sharing` (points) with `name`, `gemeinde`, `fahrzeug`, `betreiber`, `min_zoom` |
| Style | group `sharing`, off by default: point from z10, name from z14 |
| Search | category `sharing` in [Search Box](/api/search-box/) and geocoding |
| Query | [Tilequery](/api/tilequery/) with `sharing` as tileset |

The source file has one location of the Leoben station "Waasenbrücke" with a latitude about 40 km off the city; it was not taken over.
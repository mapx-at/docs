# EV charging stations

Charging stations in Austria as vector tiles (tileset ladestationen), search category ladestation and basis of EV routing.

> The E-Control terms of use do not allow passing the directory on to third parties as a web service. Tileset, search and EV routing are therefore switched off until this is settled.

Locations of public charging stations from the E-Control charging station directory. The directory is fetched hourly and mapx builds its own tileset from it every day.

> Data source: E-Control, charging station directory (national charge point register).

| | |
|---|---|
| Tileset | `ladestationen`, vector, z7 to z14 |
| Layer | `ladestationen` (points) |
| Fields | `id`, `name`, `operator`, `city`, `points` (number of charge points), `max_kw` (highest power), `connectors` (comma separated), `min_zoom`, `sort_rank` |
| Style | group `chargers`, off by default: points coloured by power (below 50 kW, from 50 kW, from 150 kW), kW label from z13 |
| Search | [Search Box](/en/api/searchbox/) and geocoding: category `ladestation` |
| Query | [Tilequery](/en/api/tilequery/) with `ladestationen` as tileset |
| Routing | [EV routing](/en/api/routing/#ev-routing) picks charging stops from the same data |

Zoom levels: stations from 150 kW appear from z7, from 50 kW from z10, the rest from z12 (field `min_zoom`).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/tilequery/ladestationen/15.33,48.23.json?radius=2000&limit=5"`} />
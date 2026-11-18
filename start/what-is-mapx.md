# What is mapx? Maps API for Austria

Source: https://docs.mapx.at/start/what-is-mapx/

> mapx is a set of maps and location APIs for Austria built only on official open data: tiles, geocoding, routing, elevation, transit and more.

mapx is a set of maps and location APIs for Austria. All data comes from official open sources (BEV, GIP.at, ASFINAG, GeoSphere Austria, ÖBB, Copernicus Sentinel-2 and others; full list under [Sources and licenses](https://docs.mapx.at/reference/attribution.md)), so there is no usage-based pricing. A free account and API token are required ([Access and limits](https://docs.mapx.at/start/access-and-limits.md)).

| Product | What you get | Start |
|---|---|---|
| Maps | Vector and raster tiles, ready-made styles (`streets` with the time-of-day variants `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `satellite-streets`, `outdoors`, `traffic-day`; list under [Styles](https://docs.mapx.at/api/styles.md)), TileJSON | [Maps](https://docs.mapx.at/guides/maps.md) |
| Search | Geocoding, type-ahead Search Box, address autofill and validation | [Search](https://docs.mapx.at/guides/search.md) |
| Navigation | Routes for car, truck, bike and foot, isochrones, matrices, map matching, tolls | [Navigation](https://docs.mapx.at/guides/navigation.md) |
| Traffic | Incident reports, motorway speed limits, a traffic layer | [Traffic](https://docs.mapx.at/guides/traffic.md) |
| Transit | Public transport journeys, stops, departures | [Transit](https://docs.mapx.at/guides/transit.md) |
| Static images | PNG or WebP maps with pins, paths and GeoJSON | [Static images](https://docs.mapx.at/guides/static-images.md) |
| Elevation and terrain | Heights, elevation profiles, terrain and surface models | [Elevation](https://docs.mapx.at/guides/elevation.md) |
| Parcels | Cadastral parcel with area and number at any point | [Parcels](https://docs.mapx.at/guides/parcels.md) |

The APIs follow the conventions of well-known web mapping services: the same URL shapes, GeoJSON responses and MapLibre-compatible styles. Existing MapLibre, Leaflet or OpenLayers code needs few changes.

## Next

- [Access and limits](https://docs.mapx.at/start/access-and-limits.md)
- [First map in 5 minutes](https://docs.mapx.at/start/first-map.md)
- [API reference](https://docs.mapx.at/api/index.md)

# Examples: runnable map code

Source: https://docs.mapx.at/examples/

> 27 runnable examples with MapLibre GL JS and the mapx-gl SDK: maps, search, navigation, elevation, transit, traffic, static images, parcels, weather.

Each example runs live on the page, shows the complete code and a single HTML file to copy. The basis is MapLibre GL JS (6.18.0) with pmtiles (4.5.0), a free token and no build. The live maps use the docs token; in your copy replace `YOUR_TOKEN` with your own public token. Filter by product:

- [Embed a map](https://docs.mapx.at/examples/embed-map.md): A map with the ready-made mapx style in an HTML page.
- [Switch the basemap](https://docs.mapx.at/examples/switch-basemap.md): Switch between map, aerial imagery and terrain.
- [Time of day and night mode](https://docs.mapx.at/examples/time-of-day.md): The default style in four lighting moods.
- [3D terrain](https://docs.mapx.at/examples/terrain-3d.md): Aerial imagery over the terrain model, tilted and exaggerated.
- [Elevation on click](https://docs.mapx.at/examples/elevation-on-click.md): A click queries the elevation at a point, terrain or surface.
- [Elevation profile](https://docs.mapx.at/examples/elevation-profile.md): Click a line, see the profile.
- [Search](https://docs.mapx.at/examples/search.md): Search for places, addresses and peaks and fly to them.
- [Route](https://docs.mapx.at/examples/route.md): Click start and destination, get the way with length and duration.
- [Weather overlay](https://docs.mapx.at/examples/weather-overlay.md): Temperature, precipitation or wind as an overlay, hour chosen with a slider.
- [Precipitation radar](https://docs.mapx.at/examples/radar-animation.md): Rain and snow of the next three hours in 15-minute steps, played as an animation.
- [Reachability (isochrones)](https://docs.mapx.at/examples/isochrone.md): Which area can be reached in 5, 10 and 15 minutes?
- [Travel-time matrix](https://docs.mapx.at/examples/matrix.md): Travel times between several places in one request.
- [Cadastre: parcel on click](https://docs.mapx.at/examples/parcel-on-click.md): Parcel boundaries over aerial imagery, a click shows the number.
- [TileJSON sources](https://docs.mapx.at/examples/tilejson-source.md): Add orthophoto and contours without a ready-made style.
- [3D buildings](https://docs.mapx.at/examples/buildings-3d.md): Extruded buildings with BEV heights, camera tilted.
- [Search Box with suggestions](https://docs.mapx.at/examples/search-box.md): Suggestions while typing, position after the click, type filter.
- [Address autofill in a form](https://docs.mapx.at/examples/autofill.md): Type a street, pick a suggestion, the form fills itself.
- [Address validation](https://docs.mapx.at/examples/address-validation.md): Check an address against the official register and show the corrected result.
- [Reverse geocoding](https://docs.mapx.at/examples/reverse-geocode.md): Click the map, get the nearest address.
- [Truck route with options](https://docs.mapx.at/examples/truck-route.md): Vehicle size, avoid tolls and motorways, alternatives.
- [Tour planning](https://docs.mapx.at/examples/optimization.md): Click stops, get the best order from the depot and back.
- [Public transit route](https://docs.mapx.at/examples/transit-route.md): Click start and destination, get walking and riding legs.
- [Departures at a stop](https://docs.mapx.at/examples/departures.md): Click near a stop, see the next departures.
- [Traffic flow and incidents](https://docs.mapx.at/examples/traffic.md): Traffic style plus the active incidents of the visible area.
- [Static image for a listing](https://docs.mapx.at/examples/static-image.md): Click a place, get a ready-made image URL with a pin.
- [Your own GeoJSON on the map](https://docs.mapx.at/examples/geojson-overlay.md): Points, line and area over the mapx style, popup on click.
- [SDK: route planner and layers](https://docs.mapx.at/examples/sdk-controls.md): RouteControl with truck profile, LayerControl, search and style switcher.

## Notes

- All data comes from official sources, no OSM data. Attribution: [License](https://docs.mapx.at/reference/attribution.md).
- All examples call the endpoints under `https://api.mapx.at/v1/`, without an access token.
- Examples marked SDK use [`mapx-gl`](https://docs.mapx.at/guides/sdk/index.md).

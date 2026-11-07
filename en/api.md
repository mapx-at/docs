# API overview

Tile, TileJSON, XYZ and elevation endpoints of mapx. Open, no key, errors as RFC 9457.

Common to all endpoints: no API key, open CORS (`Access-Control-Allow-Origin: *`), errors as `application/problem+json` per
[RFC 9457](https://www.rfc-editor.org/rfc/rfc9457), version in the path (`/v1/`).

| Endpoint | Purpose |
|---|---|
| `https://tiles.mapx.at/<file>.pmtiles` | whole file via HTTP range ([Tiles](/en/api/kacheln/#pmtiles)) |
| `https://tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}` | single tile ([Tiles](/en/api/kacheln/#xyz)) |
| `https://tiles.mapx.at/v1/{tileset}.json` | [TileJSON](/en/api/tilejson/) |
| `https://api.mapx.at/v1/elevation` | [Elevation and profile](/en/api/hoehe/) |
| `https://api.mapx.at/v1/openapi.json`, `/health` | [Operations](/en/api/fehler/#health-and-openapi) |

<CardGrid>
  <LinkCard title="Tiles" href="/en/api/kacheln/" description="PMTiles and XYZ, all tilesets." />
  <LinkCard title="TileJSON" href="/en/api/tilejson/" description="Add a tileset by URL." />
  <LinkCard title="Elevation and profile" href="/en/api/hoehe/" description="Points and profiles, with test console." />
  <LinkCard title="Errors and limits" href="/en/api/fehler/" description="problem+json, 429, caching, health." />
</CardGrid>
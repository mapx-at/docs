# API-Übersicht

Kachel-, TileJSON-, XYZ- und Höhen-Endpunkte von mapx. Offen, ohne Schlüssel, Fehler als RFC 9457.

Gemeinsam für alle Endpunkte: kein API-Schlüssel, CORS offen (`Access-Control-Allow-Origin: *`), Fehler als `application/problem+json` nach
[RFC 9457](https://www.rfc-editor.org/rfc/rfc9457), Version im Pfad (`/v1/`).

| Endpunkt | Zweck |
|---|---|
| `https://tiles.mapx.at/<datei>.pmtiles` | ganze Datei per HTTP-Range ([Kacheln](/api/kacheln/#pmtiles)) |
| `https://tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}` | einzelne Kachel ([Kacheln](/api/kacheln/#xyz)) |
| `https://tiles.mapx.at/v1/{tileset}.json` | [TileJSON](/api/tilejson/) |
| `https://api.mapx.at/v1/elevation` | [Höhe und Profil](/api/hoehe/) |
| `https://api.mapx.at/v1/openapi.json`, `/health` | [Betrieb](/api/fehler/#health-und-openapi) |

<CardGrid>
  <LinkCard title="Kacheln" href="/api/kacheln/" description="PMTiles und XYZ, alle Tilesets." />
  <LinkCard title="TileJSON" href="/api/tilejson/" description="Tileset per URL einbinden." />
  <LinkCard title="Höhe und Profil" href="/api/hoehe/" description="Punkte und Profile, mit Testkonsole." />
  <LinkCard title="Fehler und Limits" href="/api/fehler/" description="problem+json, 429, Caching, Health." />
</CardGrid>
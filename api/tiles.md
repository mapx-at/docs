# XYZ-Kacheln

Alle Kachelsätze als XYZ-Kacheln über /v1/tiles/{tileset}/{z}/{x}/{y}.{ext}, mit TileJSON, Caching und ETag.

Jeder Kachelsatz (Basiskarte, Kataster, Höhenlinien, DTM, DSM, Orthophoto, Flugblöcke) ist über dieselbe Adresse abrufbar, in der Form der Mapbox Tiles API. Ohne Schlüssel, CORS offen.

**GET https://api.mapx.at/v1/tiles/{tileset}/{z}/{x}/{y}.{ext}**

<ParamTable params={[
  { name: "tileset", type: "string", required: true, desc: "Kachelsatz, Namen siehe <a href=\"/api/kacheln/\">Kacheln</a> (z. B. <code>mapx-basis</code>, <code>mapx-kataster</code>, <code>hoehenlinien</code>, <code>dtm</code>, <code>dsm</code>, <code>dop-ortho</code>)." },
  { name: "z, x, y", type: "integer", required: true, desc: "Zoom, Spalte, Zeile im XYZ-Schema (Ursprung oben links)." },
  { name: "ext", type: "string", required: true, desc: "Passend zum Kachelsatz: <code>pbf</code> (Alias <code>mvt</code>) für Vektor, <code>webp</code> oder <code>png</code> für Raster. Eine andere Endung ergibt <code>404</code>." },
]} />

Die Adresse `tiles.mapx.at/v1/{tileset}/{z}/{x}/{y}.{ext}` ohne `/tiles/` bleibt gültig.

## TileJSON

**GET https://api.mapx.at/v1/tiles/{tileset}.json**

Das Feld `tiles` der TileJSON zeigt auf die Kachel-Adresse oben, dazu `minzoom`, `maxzoom`, `bounds`, `center` und bei Vektor `vector_layers`. Details: <a href="/api/tilejson/">TileJSON</a>.

## Antworten

| Status | Bedeutung |
|---|---|
| `200` | Kachel. Vektorkacheln mit `Content-Encoding: gzip`, wenn der Client `Accept-Encoding: gzip` sendet, sonst entpackt. |
| `304` | `If-None-Match` passt zum `ETag`. |
| `400` | Zeile ist keine Zahl. |
| `404` | Kachelsatz unbekannt, Endung passt nicht, Zoom oder Kachel außerhalb des Bereichs, Kachel leer. Fehler als `application/problem+json`. |

`ETag` stammt aus dem Datei-Hash des Kachelsatzes, `Cache-Control: public, max-age=86400`. Eine fehlende Kachel ist kein Fehler des Dienstes (wie bei Mapbox): Clients überspringen sie.

> Für MapLibre und eigenes Hosting sind die <a href="/api/kacheln/">PMTiles-Dateien</a> effizienter, ein Aufruf liest nur die nötigen Bereiche.
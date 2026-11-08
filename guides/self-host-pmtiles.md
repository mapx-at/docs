# Self-host PMTiles


**Goal:** Run a mapx PMTiles file on your own infrastructure, for example for resilience, internal networks or your own domain, or a subset of it.

**Prerequisites:** a server, S3-compatible storage or a CDN that supports HTTP range requests. A PMTiles file is a single static file; no tile server is needed.

The files are free to download with a token ([Access and limits](/start/access-and-limits/)) at `https://tiles.mapx.at/<file>.pmtiles?access_token=YOUR_TOKEN` (file names per product under "Key figures" on the [product pages](/tilesets/)). The file name contains the cut-off date and never changes; a new release gets a new name. With attribution under [CC BY 4.0](/reference/attribution/) you may redistribute and mirror the files.


1. **Download the file.** `curl -C -` resumes an interrupted download:

   ```bash
   curl -C - -O "@@url:relief@@?access_token=YOUR_TOKEN"
   ```

   The orthophoto and the elevation models are very large files; check storage and bandwidth before downloading. For sub-areas step 2 is easier.

2. **Optional: fetch only a subset.** With the [`pmtiles`](https://github.com/protomaps/go-pmtiles) command line tool, `extract` reads only the tiles of an area from the online file using range requests:

   ```bash
   # subset by bounding box (west,south,east,north)
   pmtiles extract "@@url:relief@@?access_token=YOUR_TOKEN" salzburg.pmtiles --bbox=12.9,47.7,13.2,47.9

   # or only up to a zoom level
   pmtiles extract "@@url:relief@@?access_token=YOUR_TOKEN" overview.pmtiles --maxzoom=12 --bbox=9.5,46.3,17.2,49.1
   ```

   The output is a complete PMTiles file that you serve like any other.

3. **Serve it.** The server must answer `Range` requests with `206 Partial Content`. Example nginx:

   ```nginx
   location /tiles/ {
       root /srv;
       add_header Access-Control-Allow-Origin "*" always;
       add_header Access-Control-Allow-Headers "Range, If-Match, If-None-Match" always;
       add_header Access-Control-Expose-Headers "ETag, Content-Length, Content-Range" always;
       add_header Cache-Control "public, max-age=31536000, immutable";
       if ($request_method = OPTIONS) { return 204; }
   }
   ```

   For S3 and compatible stores, a CORS rule is enough that allows `GET` and `HEAD` for your page origin (or `*`), allows the headers `Range`, `If-Match` and `If-None-Match`, and exposes `ETag`, `Content-Length`, `Content-Range`. Behind a CDN, the CDN must forward range requests to the origin or handle them itself.

4. **Use it in the map.** Your own URL instead of `tiles.mapx.at`:

   ```js
   import { Protocol } from "pmtiles";
   maplibregl.addProtocol("pmtiles", new Protocol().tile);
   // source:
   { type: "raster-dem", url: "pmtiles://https://tiles.example.com/salzburg.pmtiles", tileSize: 512, encoding: "mapbox" }
   ```

5. **Verify.** A range request must return `206` and `Content-Range`:

   ```bash
   curl -s -D - -o /dev/null -r 0-16383 https://tiles.example.com/salzburg.pmtiles
   ```


## Caching

Because file names are immutable, set `Cache-Control: public, max-age=31536000, immutable`. A new release is published under a new name; your page switches by entering the new URL. Details on caching of the mapx services: [Errors, limits and operation](/api/errors/).

## Result

A file or subset is served independently of mapx, with the same look and accuracy as the original tiles.

> An alternative without your own files: the [TileJSON address](/api/tiles/) `https://tiles.mapx.at/v1/tiles/{tileset}.json`. It always points to the current release.


## Next steps

- File format and protocol: [PMTiles documentation](https://docs.protomaps.com/pmtiles/).
- Display with MapLibre: [First map](/start/first-map/).
- Do not forget the credit: [Attribution](/reference/attribution/).


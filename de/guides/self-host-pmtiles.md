# PMTiles selbst hosten

PMTiles-Dateien herunterladen oder spiegeln und mit Range-Requests, CORS und Caching vom eigenen Server, S3 oder CDN ausliefern.

export const dtm = `https://tiles.mapx.at/${data.tilesets.relief.datei}`;

**Ziel:** Eine mapx-PMTiles-Datei auf eigener Infrastruktur betreiben, zum Beispiel für Ausfallsicherheit, interne Netze oder eigene Domain, oder einen Ausschnitt davon.

**Voraussetzungen:** ein Server, S3-kompatibler Speicher oder CDN mit Unterstützung für HTTP-Range-Anfragen. Eine PMTiles-Datei ist eine einzelne statische Datei; ein Tile-Server ist nicht nötig.

Die Dateien liegen unter `https://tiles.mapx.at/<datei>.pmtiles` (Token als `?access_token=DEIN_TOKEN` anhängen, siehe [Zugang und Limits](/de/start/access-and-limits/); Dateinamen je Produkt unter „Kennwerte“ auf den [Produktseiten](/de/tilesets/)). Der Dateiname enthält den Stichtag und ändert sich nie, ein neuer Stand bekommt einen neuen Namen. Mit Quellenangabe nach [CC BY 4.0](/de/reference/attribution/) dürfen die Dateien weitergegeben und gespiegelt werden.

<Steps>

1. **Datei herunterladen.** `curl -C -` setzt einen abgebrochenen Download fort:

   <Code lang="bash" code={`curl -C - -O "${dtm}?access_token=DEIN_TOKEN"`} />

   Das Orthophoto und die Höhenmodelle sind sehr umfangreiche Dateien; vor dem Download Speicher und Bandbreite prüfen. Für Teilgebiete geht es einfacher mit Schritt 2.

2. **Optional: nur einen Ausschnitt holen.** Mit dem Kommandozeilenwerkzeug [`pmtiles`](https://github.com/protomaps/go-pmtiles) liest `extract` per Range-Request nur die Kacheln eines Gebiets aus der Online-Datei:

   <Code lang="bash" code={`# Ausschnitt per Bounding-Box (west,süd,ost,nord)
pmtiles extract "${dtm}?access_token=DEIN_TOKEN" salzburg.pmtiles --bbox=12.9,47.7,13.2,47.9

# oder nur bis zu einer Zoomstufe
pmtiles extract "${dtm}?access_token=DEIN_TOKEN" uebersicht.pmtiles --maxzoom=12 --bbox=9.5,46.3,17.2,49.1`} />

   Die Ausgabedatei ist eine vollständige PMTiles-Datei, die sich wie jede andere ausliefern lässt.

3. **Ausliefern.** Der Server muss `Range`-Anfragen mit `206 Partial Content` beantworten. Beispiel nginx:

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

   Bei S3 und kompatiblen Speichern genügt eine CORS-Regel, die `GET` und `HEAD` für die Herkunft der eigenen Seite (oder `*`) erlaubt, die Header `Range`, `If-Match` und `If-None-Match` zulässt und `ETag`, `Content-Length`, `Content-Range` freigibt. Hinter einem CDN muss das CDN Range-Anfragen an den Ursprung weiterreichen oder selbst bedienen.

4. **In der Karte verwenden.** Statt `tiles.mapx.at` die eigene URL:

   ```js
   import { Protocol } from "pmtiles";
   maplibregl.addProtocol("pmtiles", new Protocol().tile);
   // source:
   { type: "raster-dem", url: "pmtiles://https://tiles.example.com/salzburg.pmtiles", tileSize: 512, encoding: "mapbox" }
   ```

5. **Prüfen.** Ein Range-Request muss `206` und `Content-Range` liefern:

   ```bash
   curl -s -D - -o /dev/null -r 0-16383 https://tiles.example.com/salzburg.pmtiles
   ```

</Steps>

## Caching

Weil Dateinamen unveränderlich sind, ist `Cache-Control: public, max-age=31536000, immutable` richtig. Ein neuer Stand wird unter neuem Namen veröffentlicht; die eigene Seite wechselt, indem sie die neue URL einträgt. Details zum Caching der mapx-Dienste: [Fehler, Limits und Betrieb](/de/api/errors/).

## Ergebnis

Eine Datei oder ein Ausschnitt wird unabhängig von mapx ausgeliefert, mit demselben Aussehen und derselben Genauigkeit wie die Originalkacheln.

> Eine Alternative ohne eigene Dateien: die [TileJSON-Adresse](/de/api/tiles/) `https://tiles.mapx.at/v1/tiles/{tileset}.json`. Sie zeigt immer auf den aktuellen Stand.

## Nächste Schritte

- Dateiformat und Protokoll: [PMTiles-Dokumentation](https://docs.protomaps.com/pmtiles/).
- Anzeigen mit MapLibre: [Schnellstart](/de/start/).
- Quellenangabe nicht vergessen: [Attribution](/de/reference/attribution/).
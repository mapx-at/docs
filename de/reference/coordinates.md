# Koordinaten

Reihenfolge lon,lat, WGS84 und Web-Mercator, amtliche Quellen in MGI und ETRS89.

## Reihenfolge: Länge vor Breite

Alle mapx-Schnittstellen, TileJSON, MapLibre und GeoJSON verwenden **`lon, lat`** (Länge, dann Breite), also Ost-West vor Nord-Süd:

```
12.694,47.074      // lon,lat: Großglockner-Gebiet
```

Viele Karten-Apps und GPS-Geräte zeigen dagegen `lat, lon`. Vertauschte Werte landen im Indischen Ozean, nicht in Österreich. Die [Höhen-API](/de/api/elevation/) erwartet `points=lon,lat;lon,lat`. Leaflet nimmt `[lat, lon]` bei `L.latLng`, MapLibre und OpenLayers `[lon, lat]`.

## EPSG:4326 und EPSG:3857

| | EPSG:4326 (WGS84) | EPSG:3857 (Web-Mercator) |
|---|---|---|
| Einheit | Grad | Meter |
| Verwendung | Eingabe und Ausgabe der APIs, GeoJSON, GPS | Kachelschema der Karten |
| Beispiel | `12.694, 47.074` | `1413090, 5954161` |

Eingaben an die APIs sind immer **WGS84 (EPSG:4326)**. Kacheln liegen in **Web-Mercator (EPSG:3857)**, siehe [Kachelschema](/de/reference/tile-scheme/). Clients rechnen dazwischen selbst um; die Kachelkoordinaten müssen nicht berechnet werden.

Maße in Web-Mercator sind in Österreich um rund 50 % größer als wahre Meter (Faktor `1 / cos(Breite)`). Längen und Flächen daher nicht in EPSG:3857 messen, sondern geodätisch oder in einem lokalen System wie EPSG:31287 (MGI / Austria Lambert) bzw. EPSG:3035 (ETRS89-LAEA).

## Amtliche Quellen: MGI und ETRS89

Die Ausgangsdaten der Behörden liegen in unterschiedlichen Systemen: Österreichische Landesvermessung (MGI, zum Beispiel Gauß-Krüger M28/M31/M34 und Lambert) und ETRS89 (zum Beispiel ETRS89-LAEA, UTM). Zwischen MGI und WGS84/ETRS89 liegt ein Lageunterschied von mehreren Metern, der mit einem amtlichen Transformationsgitter korrigiert wird.

**mapx transformiert korrekt**: Alle Produkte sind mit dem amtlichen Transformationsverfahren in WGS84 bzw. Web-Mercator umgerechnet. Die Ebenen liegen deckungsgleich, ohne dass eine eigene Transformation nötig ist. Eigene amtliche Daten in MGI mit dem Transformationsgitter der Software darüberlegen (zum Beispiel QGIS mit dem BEV-Gitter), sonst verschieben sie sich um einige Meter.

## Weiter

- [Kachelschema](/de/reference/tile-scheme/)
- [Höhe und Profil](/de/api/elevation/)
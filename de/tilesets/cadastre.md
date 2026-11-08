# Kataster


Die Grenzen aller Grundstücke als Linien, mit Katastralgemeinde-Nummer und Grundstücksnummer, aus der Digitalen Katastralmappe des BEV. Das Tileset hat nur z16; Clients zoomen darüber auf.



## Kennwerte



## Live-Karte



## Einbinden

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/cadastre.json" } },
        layers: [{ id: "parcel_l", type: "line", source: "v", "source-layer": "parcel_l", paint: { "line-color": "#b5651d" } }]
      }
    });
    ```
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://https://tiles.mapx.at/mapx-kataster-20260401-e4504d72.pmtiles"` (Protokoll wie im [Schnellstart](/de/start/)).
    **QGIS**

    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/cadastre/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/cadastre.json
    curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/cadastre/14/8860/5730.pbf
    ```
  
Weil es nur z16 gibt, im Style `minzoom: 16` setzen; die Quelle mit `maxzoom: 16` überzoomt automatisch.

## Layer-Referenz

Die Linien sind Grenzen, keine Flächen: Ein Klick trifft die Grenzlinie und liefert das Grundstück, zu dem sie gehört. Eigentümer und Fläche sind nicht enthalten.

### parcel_l



## Quelle und Aktualität

- Quelle: BEV **Digitale Katastralmappe (DKM)**, Stichtag 01.04.2026.

## Hinweise

> Die Darstellung ist nicht rechtsverbindlich. Für Grenzfragen gilt der Kataster beim Vermessungsamt.


- Die Abbildung der Grenzen ist nicht rechtsverbindlich, maßgeblich ist der Kataster beim Vermessungsamt.
- Unter z16 gibt es keine Kacheln.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

{{< attribution ts="kataster" >}}

Mehr: [Lizenz](/de/reference/attribution/).


# Basiskarte


Die Basiskarte vereint die offenen Daten von BEV, GIP, Umweltbundesamt und BMLUK in 18 Layern. Sie enthält keine OpenStreetMap-Daten. Kataster, Höhenlinien und Relief sind eigene Produkte und lassen sich darüberlegen.



## Kennwerte



## Live-Karte



Die Farben der Live-Karte sind automatisch vergeben (Fläche, Linie, Punkt je Layer), kein fertiger Kartenstil. Klick zeigt die Attribute.

## Einbinden

  **MapLibre**

    ```js
    new maplibregl.Map({
      container: "map",
      style: {
        version: 8,
        sources: { v: { type: "vector", url: "https://tiles.mapx.at/v1/tiles/streets.json" } },
        layers: [{ id: "road_l", type: "line", source: "v", "source-layer": "road_l", paint: { "line-color": "#b5651d" } }]
      }
    });
    ```
    TileJSON liefert Zoombereich, Grenzen und Attribution; direkt per PMTiles geht `url: "pmtiles://https://tiles.mapx.at/mapx-basis-20260401-6ae3c128.pmtiles"` (Protokoll wie im [Schnellstart](/de/start/)).
    **QGIS**

    Vektorkacheln → Neue Verbindung, URL `https://tiles.mapx.at/v1/tiles/streets/{z}/{x}/{y}.pbf`, Stil-URL leer lassen. Alternativ die PMTiles-Datei direkt als Layer öffnen (QGIS ab 3.32 liest PMTiles per HTTP).
    **curl**

    ```bash
    curl -s -H "Authorization: Bearer DEIN_TOKEN" https://tiles.mapx.at/v1/tiles/streets.json
    curl -s -H "Authorization: Bearer DEIN_TOKEN" -o kachel.pbf https://tiles.mapx.at/v1/tiles/streets/14/8860/5730.pbf
    ```
  
## Layer-Referenz

Layername endet auf `_a` (Fläche), `_l` (Linie) oder `_p` (Punkt). `min_zoom` am Objekt sagt, ab welcher Stufe es gedacht ist; `sort_rank` ordnet innerhalb des Layers.



Alle Felder und Werte jedes Layers: [Basiskarte: Layer-Referenz](/de/reference/style-spec/).

## Quelle und Aktualität

| Thema | Quelle | Stand |
|---|---|---|
| Nutzung, Gebäude | BEV Digitale Katastralmappe (DKM), Höhen aus dem DLM | 01.04.2026 |
| Straßen, Bahn | GIP-OGD (Graphenintegrationsplattform) | 30.04.2026 |
| Lifte, Stationen, POI | BEV DLM, Wien-OGD (nur Wien) | |
| Grenzen | BEV Verwaltungsgrenzen (VGD) | 01.10.2026 |
| Orte, Namen, Gipfel | BEV Geographische Namen | 23.03.2026 |
| Hausnummern | BEV Adressregister | 01.10.2026 |
| Gewässer | BMLUK Gesamtgewässernetz (GGN v19) | |
| Skigebiete | Umweltbundesamt | 2017 |
| Flugplätze | BEV DLM Verkehr (CC BY 4.0) | |


## Hinweise

- Nicht enthalten: Halte von Privatbahnen sowie U-Bahn- und Straßenbahnhaltestellen; kleine Flugfelder fehlen teils. Skigebiete haben den Stand 2017.
- Ausland: praktisch leer (nur Bodensee und Gewässer bis knapp hinter die Grenze).
- Glyphen und Schriften für Beschriftungen liefert mapx nicht; der Style braucht eigene.

## Attribution

Pflicht nach CC BY 4.0. MapLibre übernimmt den Text automatisch aus TileJSON oder PMTiles:

{{< attribution ts="basis" >}}

Mehr: [Lizenz](/de/reference/attribution/).


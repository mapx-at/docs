# Beispiele: lauffähiger Karten-Code

Source: https://docs.mapx.at/de/examples/

> 27 lauffähige Beispiele mit MapLibre GL JS und dem SDK mapx-gl: Karten, Suche, Navigation, Höhe, ÖV, Verkehr, statische Bilder, Grundstücke, Wetter.

Jedes Beispiel läuft live auf der Seite, zeigt den vollständigen Code und eine einzelne HTML-Datei zum Kopieren. Grundlage sind MapLibre GL JS (6.18.0) mit pmtiles (4.5.0), mit kostenlosem Token und ohne Build. Die Live-Karten nutzen den Token der Doku; in der Kopie `YOUR_TOKEN` durch den eigenen öffentlichen Token ersetzen. Nach Produkt filtern:

- [Karte einbetten](https://docs.mapx.at/de/examples/embed-map.md): Eine Karte mit dem fertigen mapx-Stil in einer HTML-Seite.
- [Basiskarte wechseln](https://docs.mapx.at/de/examples/switch-basemap.md): Zwischen Karte, Luftbild und Gelände umschalten.
- [Tageszeit und Nachtmodus](https://docs.mapx.at/de/examples/time-of-day.md): Den Standardstil in vier Lichtstimmungen.
- [3D-Gelände](https://docs.mapx.at/de/examples/terrain-3d.md): Luftbild über dem Geländemodell, geneigt und überhöht.
- [Höhe per Klick](https://docs.mapx.at/de/examples/elevation-on-click.md): Ein Klick fragt die Höhe an einem Punkt, wahlweise Gelände oder Oberfläche.
- [Höhenprofil](https://docs.mapx.at/de/examples/elevation-profile.md): Linie klicken, Profil sehen.
- [Suche](https://docs.mapx.at/de/examples/search.md): Orte, Adressen und Gipfel suchen und anspringen.
- [Route](https://docs.mapx.at/de/examples/route.md): Start und Ziel klicken, Weg samt Länge und Dauer erhalten.
- [Wetter-Overlay](https://docs.mapx.at/de/examples/weather-overlay.md): Temperatur, Niederschlag oder Wind als Overlay, Stunde per Regler.
- [Niederschlagsradar](https://docs.mapx.at/de/examples/radar-animation.md): Regen und Schnee der nächsten drei Stunden in 15-Minuten-Schritten als Animation.
- [Erreichbarkeit (Isochronen)](https://docs.mapx.at/de/examples/isochrone.md): Welche Fläche ist in 5, 10 und 15 Minuten erreichbar?
- [Fahrzeitmatrix](https://docs.mapx.at/de/examples/matrix.md): Fahrzeiten zwischen mehreren Orten in einer Anfrage.
- [Kataster: Grundstück per Klick](https://docs.mapx.at/de/examples/parcel-on-click.md): Grundstücksgrenzen über dem Luftbild, ein Klick nennt die Nummer.
- [TileJSON-Quellen](https://docs.mapx.at/de/examples/tilejson-source.md): Orthophoto und Höhenlinien ohne fertigen Stil einbinden.
- [3D-Gebäude](https://docs.mapx.at/de/examples/buildings-3d.md): Gebäude mit BEV-Höhen als 3D-Körper, Kamera geneigt.
- [Search Box mit Vorschlägen](https://docs.mapx.at/de/examples/search-box.md): Vorschläge beim Tippen, Position nach dem Klick, Typfilter.
- [Adress-Autofill im Formular](https://docs.mapx.at/de/examples/autofill.md): Straße tippen, Vorschlag wählen, das Formular füllt sich.
- [Adressvalidierung](https://docs.mapx.at/de/examples/address-validation.md): Eine Adresse gegen das amtliche Register prüfen und das korrigierte Ergebnis zeigen.
- [Reverse Geocoding](https://docs.mapx.at/de/examples/reverse-geocode.md): Karte anklicken, nächste Adresse erhalten.
- [Lkw-Route mit Optionen](https://docs.mapx.at/de/examples/truck-route.md): Fahrzeugmaße, Maut und Autobahn meiden, Alternativen.
- [Tourenplanung](https://docs.mapx.at/de/examples/optimization.md): Haltestellen anklicken, beste Reihenfolge ab Depot und zurück.
- [ÖV-Verbindung](https://docs.mapx.at/de/examples/transit-route.md): Start und Ziel klicken, Fuß- und Fahrabschnitte erhalten.
- [Abfahrten an einer Haltestelle](https://docs.mapx.at/de/examples/departures.md): Nahe an eine Haltestelle klicken, nächste Abfahrten sehen.
- [Verkehrslage und Meldungen](https://docs.mapx.at/de/examples/traffic.md): Verkehrsstil plus die aktiven Meldungen im sichtbaren Ausschnitt.
- [Static Image fürs Exposé](https://docs.mapx.at/de/examples/static-image.md): Ort anklicken, fertige Bild-URL mit Marker erhalten.
- [Eigenes GeoJSON auf der Karte](https://docs.mapx.at/de/examples/geojson-overlay.md): Punkte, Linie und Fläche über dem mapx-Stil, Popup per Klick.
- [SDK: Routenplaner und Ebenen](https://docs.mapx.at/de/examples/sdk-controls.md): RouteControl mit Lkw-Profil, LayerControl, Suche und Stilumschalter.

## Hinweise

- Alle Daten stammen aus amtlichen Quellen, keine OSM-Daten. Attribution: [Lizenz](https://docs.mapx.at/de/reference/attribution.md).
- Alle Beispiele rufen die Endpunkte unter `https://api.mapx.at/v1/` mit einem öffentlichen Token auf (siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)).
- Beispiele mit SDK-Kennzeichnung nutzen [`mapx-gl`](https://docs.mapx.at/de/guides/sdk/index.md).

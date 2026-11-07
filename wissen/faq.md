# Häufige Fragen

Antworten auf häufige Fragen zu Schlüssel, Lizenz, kommerzieller Nutzung, Aktualität, Offline-Nutzung, Genauigkeit und Fehlern.

## Brauche ich einen API-Schlüssel oder ein Konto?

Nein. Kacheln, TileJSON und die Höhen-API sind ohne Schlüssel und ohne Anmeldung nutzbar, CORS ist offen. Siehe [Schnellstart](/start/).

## Was kostet die Nutzung, unter welcher Lizenz stehen die Daten?

Die Nutzung ist kostenlos. Die Daten stehen unter [CC BY 4.0](/lizenz/): Sie müssen die Quelle nennen, alles Weitere ist frei. Wie die Angabe aussieht, zeigt [Attribution](/anleitungen/attribution/).

## Darf ich die Daten kommerziell nutzen?

Ja. CC BY 4.0 erlaubt kommerzielle Nutzung, auch in eigenen Produkten und Apps, solange die Quelle genannt wird.

## Wie aktuell sind die Daten?

Jedes Produkt hat einen Stichtag der amtlichen Quelle, zum Beispiel 15.09.2025 für Höhenmodelle und Höhenlinien, Oktober 2025 für das Orthophoto, 01.04.2026 für Basiskarte und Kataster. Der Stichtag steht auf der [Produktseite](/daten/), in den Kachel-Metadaten und in der Höhen-API als `stichtag`. Dateinamen enthalten ihn.

## Ändert sich eine URL, wenn es neue Daten gibt?

Die PMTiles-Dateinamen enthalten den Stichtag; ein neuer Stand bekommt einen neuen Namen, alte Dateien bleiben gültig. Wollen Sie immer den aktuellen Stand, nutzen Sie die [TileJSON](/api/tilejson/)-Adresse `https://tiles.mapx.at/v1/{tileset}.json`.

## Kann ich die Karte offline nutzen?

Ja. PMTiles sind einzelne Dateien; Sie laden eine Datei oder einen Ausschnitt herunter und liefern sie selbst aus oder lesen sie lokal. Siehe [PMTiles selbst hosten](/anleitungen/pmtiles-selbst-hosten/) und [mapx in QGIS](/anleitungen/qgis/).

## Gibt es Daten außerhalb Österreichs?

Die Produkte decken Österreich ab. Das Orthophoto und die Basiskarte reichen an den Rändern in die Nachbarländer, die genaue Abdeckung steht unter „Kennwerte“ (Gebiet) auf den Produktseiten und in `bounds` der TileJSON. Außerhalb liefern Kacheln `404`.

## Wie genau sind die Höhen?

Die Genauigkeit entspricht der amtlichen Quelle des BEV (Laserscanning, 1-m-Raster). Die Kacheln speichern Höhen in Schritten von 0,1 m. Details: [DTM Geländemodell](/daten/dtm/) und [Terrain-RGB](/konzepte/terrain-rgb/).

## Was ist der Unterschied zwischen DTM und DSM?

Das **[DTM](/daten/dtm/)** (Geländemodell) zeigt den Boden ohne Bewuchs und Gebäude. Das **[DSM](/daten/dsm/)** (Oberflächenmodell) zeigt die Oberfläche inklusive Bäumen und Bauwerken. Für Wanderwege, Gefälle und Profile nehmen Sie das DTM, für Sichtbarkeit, Solarplanung und Gebäudehöhen das DSM. In der [Höhen-API](/api/hoehe/) wählen Sie mit `model=dtm` oder `model=dsm`.

## Warum bekomme ich 404 bei einer Kachel?

Typische Gründe: Die Kachel liegt außerhalb des Datengebiets oder des Zoombereichs, die Dateiendung passt nicht zum Typ (`webp` für Raster, `pbf` für Vektor) oder der Tilesetname ist falsch. Bei Rasterkacheln deshalb `minzoom`/`maxzoom` aus der TileJSON verwenden. Siehe [Kacheln](/api/kacheln/) und [Fehler und Limits](/api/fehler/).

## Warum bekomme ich 429?

Der Dienst begrenzt Anfragen je IP (Standard 50 pro Sekunde). Warten Sie die Zeit aus dem Header `Retry-After` ab und versuchen Sie es mit wachsendem Abstand erneut. Für höheren Durchsatz hosten Sie die Dateien selbst, siehe [PMTiles selbst hosten](/anleitungen/pmtiles-selbst-hosten/). Details unter [Fehler und Limits](/api/fehler/).

## Meine Karte ist um eine Zoomstufe versetzt oder unscharf. Was tun?

Die Rasterkacheln haben 512 Pixel. Setzen Sie im Client die Kachelgröße 512, siehe [Kachelschema](/wissen/kachelschema/).

## Meine Punkte landen im falschen Land. Warum?

Vermutlich sind Länge und Breite vertauscht. mapx und GeoJSON erwarten `lon,lat`, siehe [Koordinaten](/wissen/koordinaten/).

## Sind die Katasterdaten rechtsverbindlich?

Nein. Der [Kataster](/daten/kataster/) zeigt die Grundstücksgrenzen der DKM zur Orientierung, er ersetzt keinen Auszug aus dem Grundbuch oder eine Vermessung.

## Gibt es Kartenstil, Schriften, Suche oder Routing?

mapx liefert Daten und Höhen, keinen fertigen Kartenstil, keine Schriften, keine Suche und kein Routing. Einen eigenen Stil bauen Sie mit [Basiskarte stylen](/anleitungen/basiskarte-stylen/).
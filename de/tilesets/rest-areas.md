# Rastanlagen-Tileset (ASFINAG)

Source: https://docs.mapx.at/de/tilesets/rest-areas/

> Rastanlagen und Lkw-Parkplätze der ASFINAG als Vektorkacheln (Tileset rastanlagen) und als Suchkategorie rastplatz in der mapx-Suche.

Rast-, Pendler- und Lkw-Parkplätze an Autobahnen und Schnellstraßen aus den Verkehrsdaten der ASFINAG, 314 Standorte.

> **Achtung:** **Namensnennung**
> 
> Datenquelle: ASFINAG, Rast-, Pendler- und Lkw-Parkplätze, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de). Bearbeitung: Auswahl und Umformung der Felder.

| | |
|---|---|
| Tileset | `rastanlagen`, Vektor, z7 bis z14 |
| Layer | `rastanlagen` (Punkte) mit `id`, `name`, `road`, `car` und `lorry` (Stellplätze), `free` (kostenfrei), `usage`, `min_zoom` |
| Stil | Gruppe `rest_areas`, standardmäßig aus: Punkt, ab z12 Name; Anlagen mit Lkw-Plätzen dunkler |
| Suche | Kategorie `rastplatz` in [Search Box](https://docs.mapx.at/de/api/search-box.md) und Geocoding |
| Abfrage | [Tilequery](https://docs.mapx.at/de/api/tilequery.md) mit `rastanlagen` als Tileset |

Die Belegung in Echtzeit ist (noch) nicht Teil der Kacheln.

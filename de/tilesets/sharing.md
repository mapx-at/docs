# Sharing-Standorte

Sharing-Stationen (Lastenräder, Fahrräder) der ÖBB als Vektorkacheln (Tileset sharing) und Suchkategorie sharing.

Standorte von Sharing-Stationen aus dem Datensatz „Shared-Mobility Standorte 2026“ der ÖBB-Personenverkehr AG. Der Datensatz enthält nur Standorte, die die ÖBB selbst betreibt und besitzt; derzeit sieben Stationen in Korneuburg und Leoben (Stand 07.11.2026).

> Datenquelle: ÖBB-Personenverkehr AG, Shared-Mobility Standorte, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.de).

| | |
|---|---|
| Tileset | `sharing`, Vektor, z8 bis z14 |
| Layer | `sharing` (Punkte) mit `name`, `gemeinde`, `fahrzeug`, `betreiber`, `min_zoom` |
| Stil | Gruppe `sharing`, standardmäßig aus: Punkt ab z10, Name ab z14 |
| Suche | Kategorie `sharing` in [Search Box](/de/api/search-box/) und Geocoding |
| Abfrage | [Tilequery](/de/api/tilequery/) mit `sharing` als Tileset |

Die Quelldatei hat einen Standort der Leobener Station „Waasenbrücke“ mit einem Breitengrad rund 40 km neben der Stadt; er wurde nicht übernommen.
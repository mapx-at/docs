# Radrouten-Tileset (GIP.at)

Source: https://docs.mapx.at/de/tilesets/bike-routes/

> Beschilderte Radrouten und MTB-Strecken in Österreich aus GIP.at, inklusive EuroVelo, als Vektorkacheln (Tileset bike-routes) für MapLibre.

Radrouten und Mountainbike-Strecken aus dem offenen Datensatz der Graphenintegrations-Plattform (GIP.at, GIPOGD), inklusive der EuroVelo-Routen. Im Stil `streets` stecken sie in der Ebenengruppe `bike_routes` (standardmäßig aus); siehe [Tourismus](https://docs.mapx.at/de/use-cases/tourism.md).

> **Achtung:** **Namensnennung**
> 
> Datenquelle: GIP.at, CC BY 4.0

| | |
|---|---|
| Tileset | `bike-routes`, Vektor, z5 bis z14, Stand 20260708 |
| TileJSON | `https://api.mapx.at/v1/tiles/bike-routes.json` |
| Layer | `bike_routes` (Linien) mit `name`, `ref`, `network`, `type`, `difficulty`, `signed`, `min_zoom` |
| Stil | Gruppe `bike_routes` in `streets`, standardmäßig aus |
| Abfrage | [Tilequery](https://docs.mapx.at/de/api/tilequery.md) mit `bike-routes` als Tileset |

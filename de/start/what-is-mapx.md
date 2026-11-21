# Was ist mapx? Karten-API für Österreich

Source: https://docs.mapx.at/de/start/what-is-mapx/

> mapx ist eine Sammlung von Karten- und Location-APIs für Österreich, ausschließlich auf amtlichen offenen Daten: Kacheln, Geocoding, Routing, Höhe, ÖV und mehr.

mapx ist eine Sammlung von Karten und Location-APIs für Österreich. Alle Daten stammen aus offiziellen offenen Quellen (BEV, GIP.at, ASFINAG, GeoSphere Austria, ÖBB, Copernicus Sentinel-2 und weitere; vollständig unter [Quellen und Lizenzen](https://docs.mapx.at/de/reference/attribution.md)). Ein kostenloses Konto und ein API-Token sind nötig ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)).

| Produkt | Inhalt | Einstieg |
|---|---|---|
| Karten | Vektor- und Rasterkacheln, fertige Stile (`streets` mit den Tageszeit-Varianten `streets-dawn`, `streets-dusk`, `streets-night`, `satellite`, `satellite-current` (Sentinel-2, aktuelles Satellitenbild), `satellite-streets`, `outdoors`, `outdoors-winter`, `light`, `dark`, `traffic-day`, `traffic-night`, die Reliefkarten `relief`, `relief-contours`, `relief-light`, `relief-dark`, dazu 3D-Fassungen wie `outdoors-3d`, `satellite-3d` und `streets-3d`; Liste unter [Stile](https://docs.mapx.at/de/api/styles.md)), TileJSON | [Karten](https://docs.mapx.at/de/guides/maps.md) |
| Suche | Geocoding, Type-ahead-Search-Box, Adress-Autofill und Validierung | [Suche](https://docs.mapx.at/de/guides/search.md) |
| Navigation | Routen für Auto, Lkw, Rad und zu Fuß, Isochronen, Matrizen, Map Matching, Maut | [Navigation](https://docs.mapx.at/de/guides/navigation.md) |
| Verkehr | Störungsmeldungen, Tempolimits auf Autobahnen, eine Verkehrsebene | [Verkehr](https://docs.mapx.at/de/guides/traffic.md) |
| ÖV | Verbindungen im öffentlichen Verkehr, Haltestellen, Abfahrten | [ÖV](https://docs.mapx.at/de/guides/transit.md) |
| Statische Kartenbilder | PNG- oder WebP-Karten mit Pins, Pfaden und GeoJSON | [Statische Kartenbilder](https://docs.mapx.at/de/guides/static-images.md) |
| Höhen und Gelände | Höhen, Höhenprofile, Gelände- und Oberflächenmodelle | [Höhen](https://docs.mapx.at/de/guides/elevation.md) |
| Wetter und Umwelt | Messwerte, Vorhersage, Warnungen, Radar- und Wetterkacheln, Luftqualität, Klima | [Wetter](https://docs.mapx.at/de/api/weather.md) |

Die APIs folgen den Konventionen bekannter Web-Mapping-Dienste: gleiche URL-Formen, GeoJSON-Antworten und MapLibre-kompatible Stile. Bestehender Code für MapLibre, Leaflet oder OpenLayers braucht wenige Änderungen.

## Nächste Schritte

- [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)
- [Erste Karte in 5 Minuten](https://docs.mapx.at/de/start/first-map.md)
- [API-Referenz](https://docs.mapx.at/de/api/index.md)

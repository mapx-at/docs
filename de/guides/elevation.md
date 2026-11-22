# Höhen-Guide: Höhenprofil und Gelände

Source: https://docs.mapx.at/de/guides/elevation/

> Höhen an Punkten und Höhenprofile abfragen, Geländemodelle nutzen und 3D-Gelände in der Karte zeigen: Höhen-API und Gelände-Kachelsätze für Österreich.

mapx hat zwei Höhenmodelle mit 1 m Auflösung (BEV, Laserscanning): das Geländemodell `dtm` (nackter Boden) und das Oberflächenmodell `dsm` (mit Bäumen und Gebäuden). `dtm` eignet sich für Steigungen, Profile und Wege, `dsm` für Sichtbarkeit und Gebäudehöhen. Jede Anfrage braucht einen Token ([Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)); die Beispiele senden ihn im Header `Authorization`, den auch Browser senden dürfen.

## Höhe an einem Punkt

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/elevation/12.694,47.074" | jq '.properties | {dtm, dsm}'
```

Mehrere Punkte, bis zu 512, in einer Anfrage:

```js
const url = "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm";
const { features } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(features.map((f) => f.properties.elevation)); // Meter über dem Meer
```

Höhen sind in Metern mit Schritten von 0,1 m. Außerhalb des Datengebiets ist der Wert `null`.

## Höhenprofil

```js
const url = "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=50";
const { properties: p } = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
console.log(p.ascent_m, p.descent_m, p.min_m, p.max_m);
// p.distances[i] (m ab Start) und p.elevations[i] (m) sind bereit für ein Diagramm
```

Bis zu 1024 Samples entlang einer Linie mit 2 bis 512 Stützpunkten. Anstieg und Abstieg werden mit 3 m Hysterese gezählt (`ascent_threshold_m`): eine Steigung oder ein Gefälle zählt erst, wenn sich die Höhe mindestens 3 m vom letzten Wendepunkt entfernt hat. Geländerauschen summiert sich so nicht auf, und die Summen hängen kaum von `samples` ab. Mit `model=dsm` zählen Bäume und Gebäude als Gelände, der Anstieg fällt höher aus.

Eine Routengeometrie geht mit `POST /v1/elevation/profile` direkt ins Profil (GeoJSON-LineString, ein Feature oder `{"coordinates": [...]}`, bis 20000 Stützpunkte):

```js
const route = (await (await fetch("https://api.mapx.at/v1/directions/cycling/12.8418,47.0378;12.7509,47.0742?geometries=geojson&overview=full", { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json()).routes[0];
const prof = await (await fetch("https://api.mapx.at/v1/elevation/profile?samples=200", {
  method: "POST",
  headers: { Authorization: "Bearer DEIN_TOKEN", "Content-Type": "application/json" },
  body: JSON.stringify(route.geometry),
})).json();
console.log(prof.properties.ascent_m);
```

 Beispiel mit Diagramm: [Höhenprofil](https://docs.mapx.at/de/examples/elevation-profile.md). Routen tragen Anstieg und Abstieg bereits in `routes[].summary`; mit `annotations=elevation` kommt eine Höhe je Segment.

## 3D-Gelände in der Karte

Der Kachelsatz `terrain-rgb` als `raster-dem`-Source hinzufügen und das Gelände einschalten (Token-Übergabe für Karten: [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md)):

```js
map.on("load", () => {
  // zwei getrennte Quellen: eine für das Gelände, eine für die Schummerung (bessere Darstellung)
  map.addSource("dem", { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512 });
  map.addSource("dem-hillshade", { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512 });
  map.setTerrain({ source: "dem", exaggeration: 1.2 });
  map.addLayer({ id: "hillshade", type: "hillshade", source: "dem-hillshade" });
  map.setPitch(60);
});
```

Schritt für Schritt mit Luftbild: [3D-Gelände mit Orthofoto](https://docs.mapx.at/de/guides/3d-terrain.md). Der Stil `outdoors` enthält Schummerung und Höhenlinien bereits; die fertigen 3D-Stile `outdoors-3d` und `satellite-3d` bringen das Gelände mit, siehe [Karten](https://docs.mapx.at/de/guides/maps.md).

## Höhen aus Kacheln lesen

Die Kacheln verwenden die Terrain-RGB-Kodierung: `height = -10000 + (R * 65536 + G * 256 + B) * 0.1`. Für Massenarbeit die Kacheln selbst dekodieren, siehe [Terrain-RGB](https://docs.mapx.at/de/reference/terrain-rgb.md). Für einzelne Punkte ist die API einfacher.

## Nächste Schritte

- [Elevation API](https://docs.mapx.at/de/api/elevation.md)
- [Kachelsätze](https://docs.mapx.at/de/tilesets/terrain-rgb.md): Datenblatt des Geländemodells
- [Höhenlinien überlagern](https://docs.mapx.at/de/guides/overlay-contours.md)

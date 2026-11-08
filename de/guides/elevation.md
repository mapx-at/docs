# Höhen und Gelände

Höhen an Punkten, Höhenprofile, Geländemodelle und 3D-Gelände in der Karte.

mapx hat zwei Höhenmodelle mit 1 m Auflösung (BEV, Laserscanning): das Geländemodell `dtm` (nackter Boden) und das Oberflächenmodell `dsm` (mit Bäumen und Gebäuden). `dtm` eignet sich für Steigungen, Profile und Wege, `dsm` für Sichtbarkeit und Gebäudehöhen.

## Höhe an einem Punkt

```bash
curl -s "https://api.mapx.at/v1/elevation/12.694,47.074" | jq '.properties | {dtm, dsm}'
```

Mehrere Punkte, bis zu 512, in einer Anfrage:

```js
const url = "https://api.mapx.at/v1/elevation?points=12.694,47.074;15.3294,48.2272&model=dtm";
const { features } = await (await fetch(url)).json();
console.log(features.map((f) => f.properties.elevation)); // Meter über dem Meer
```

Höhen sind in Metern mit Schritten von 0,1 m. Außerhalb des Datengebiets ist der Wert `null`.

## Höhenprofil

```js
const url = "https://api.mapx.at/v1/elevation/profile?coordinates=12.69,47.07;12.72,47.09&samples=50";
const { properties: p } = await (await fetch(url)).json();
console.log(p.ascent_m, p.descent_m, p.min_m, p.max_m);
// p.distances[i] (m ab Start) und p.elevations[i] (m) sind bereit für ein Diagramm
```

Bis zu 1024 Samples entlang einer Linie mit 2 bis 512 Stützpunkten. Beispiel mit Diagramm: [Höhenprofil](/de/examples/elevation-profile/). Routen tragen Anstieg und Abstieg bereits in `routes[].summary`; mit `annotations=elevation` kommt eine Höhe je Segment.

## 3D-Gelände in der Karte

Das Tileset `terrain-rgb` als `raster-dem`-Source hinzufügen und das Gelände einschalten:

```js
map.on("load", () => {
  map.addSource("dem", { type: "raster-dem", url: "https://tiles.mapx.at/v1/tiles/terrain-rgb.json", tileSize: 512 });
  map.setTerrain({ source: "dem", exaggeration: 1.2 });
  map.addLayer({ id: "hillshade", type: "hillshade", source: "dem" });
});
map.setPitch(60);
```

Schritt für Schritt mit Luftbild: [3D-Gelände mit Orthophoto](/de/guides/3d-terrain/). Der Style `outdoors` enthält Schummerung und Höhenlinien bereits.

## Höhen aus Tiles lesen

Die Tiles verwenden die Terrain-RGB-Kodierung: `height = -10000 + (R * 65536 + G * 256 + B) * 0.1`. Für Massenarbeit die Tiles selbst dekodieren, siehe [Terrain-RGB](/de/reference/terrain-rgb/). Für einzelne Punkte ist die API einfacher.

## Nächste Schritte

- [Elevation API](/de/api/elevation/)
- [Tilesets](/de/tilesets/terrain-rgb/): Datenblatt des Geländemodells
- [Höhenlinien überlagern](/de/guides/overlay-contours/)
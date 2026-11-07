# Attribution korrekt anzeigen

Quellenangabe nach CC BY 4.0 in Web-Karten, Druck und statischen Bildern: Text je Produkt und automatische Anzeige in MapLibre.

export const t = data.tilesets;
export const s = (h) => (h || "").replace(/<[^>]+>/g, "");

**Ziel:** Die nach [CC BY 4.0](/lizenz/) verlangte Quellenangabe überall dort zeigen, wo mapx-Daten sichtbar sind.

Die Pflicht: Quelle nennen, Lizenz nennen (CC BY 4.0) und keine Änderung als amtlich ausgeben. Die Angabe muss für Nutzer der Karte erkennbar sein, etwa in der Ecke der Karte, nicht nur im Impressum.

## Web-Karte mit MapLibre

Binden Sie das Tileset über [TileJSON](/api/tilejson/) oder PMTiles ein, übernimmt MapLibre den Text automatisch in das Attributionselement der Karte (unten rechts, „i“-Symbol). Sie müssen nichts hinzufügen, solange Sie es nicht ausblenden:

```js
sources: { dem: { type: "raster-dem", url: "https://tiles.mapx.at/v1/dtm.json" } }
// Attribution erscheint automatisch
```

Mehrere Quellen werden zusammengeführt. Haben Sie `attributionControl: false` gesetzt oder eine Quelle ohne TileJSON mit `tiles: [...]` angelegt, tragen Sie den Text selbst ein:

```js
{ type: "raster", tiles: ["https://tiles.mapx.at/v1/orthophoto/{z}/{x}/{y}.webp"], tileSize: 512,
  maxzoom: 18, attribution: "Orthophoto: BEV (CC BY 4.0)" }
```

Bei Leaflet und OpenLayers geben Sie den Text über `attribution` bzw. `attributions` der Quelle an, siehe Einbinden-Beispiele auf den [Produktseiten](/daten/).

## Druck, Berichte, statische Bilder

Dort gibt es kein Attributionselement. Setzen Sie die Quellenangabe als sichtbare Zeile ins Bild oder an die Karte, in lesbarer Größe:

```
Quelle: BEV – ALS-DTM 1 m (Stichtag 15.09.2025), CC BY 4.0
```

Werden mehrere Produkte kombiniert, listen Sie alle auf, getrennt durch Strichpunkt. In einem Bericht mit Karte genügt die Zeile unter der Abbildung.

## Text je Produkt

| Produkt | Text |
|---|---|
| [DTM](/daten/dtm/) | {s(t.relief.metadata.attribution)} |
| [DSM](/daten/dsm/) | {s(t.dsm.metadata.attribution)} |
| [Orthophoto](/daten/orthophoto/) | {s(t.dop.metadata.attribution)} |
| [Basiskarte](/daten/basiskarte/) | {s(t.basis.metadata.attribution)} |
| [Kataster](/daten/kataster/) | {s(t.kataster.metadata.attribution)} |
| [Höhenlinien](/daten/hoehenlinien/) | {s(t.hoehenlinien.metadata.attribution)} |
| [Flugblöcke](/daten/flugbloecke/) | BEV, CC BY 4.0 |

Die Texte sind auch in den TileJSON- und PMTiles-Metadaten hinterlegt. Mit HTML-Link zur Quelle finden Sie sie auf den Produktseiten unter „Attribution“.

> Zeigen Sie die Angabe in der Karte selbst und zusätzlich in Ihren Nutzungshinweisen. Den kompletten Lizenztext finden Sie unter [Lizenz](/lizenz/).

## Nächste Schritte

- Lizenz im Detail: [Lizenz](/lizenz/).
- Häufige Fragen zu Lizenz und kommerzieller Nutzung: [FAQ](/wissen/faq/).
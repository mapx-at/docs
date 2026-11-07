# Tilequery und Grundstück

Vektor-Features im Umkreis eines Punkts mit Abstand (wie Mapbox Tilequery), dazu Grundstück und Höhe an einem Punkt.

Tilequery beantwortet „Was liegt an diesem Punkt?“, ohne Kacheln selbst zu laden und zu zerlegen. Der Server liest die Vektorkachel auf der höchsten Zoomstufe des Tilesets (bei Radius auch die Nachbarkacheln), berechnet den Abstand in Metern und liefert GeoJSON im Stil von Mapbox. Kein Schlüssel, CORS offen.

## Tilequery

**GET https://api.mapx.at/v1/tilequery/{tileset}/{lon},{lat}.json**

`tileset` ist ein Name aus der [Kachel-Liste](/api/kacheln/) (zum Beispiel `streets`, `cadastre`, `contours`; die früheren Namen `basis`, `kataster`, `hoehenlinien` gelten bis 31.05.2027 als Alias, siehe [Migration](/api/migration/)). Mehrere Tilesets gehen kommagetrennt (höchstens 15): `streets,cadastre`. Alle Namen liefert `GET https://api.mapx.at/v1/tiles` ([Tileset-Liste](/api/tiles/#tileset-liste)).

<ParamTable params={[
  { name: "radius", type: "number", default: "0", desc: "Umkreis in Metern, 0 bis 2000. Bei 0 kommen nur Features, die den Punkt enthalten (Polygone)." },
  { name: "limit", type: "integer", default: "5", desc: "Anzahl Treffer, 1 bis 50." },
  { name: "layers", type: "string", desc: "Kommagetrennte Ebenennamen, zum Beispiel <code>building_a,road_l</code>." },
  { name: "geometry", type: "string", desc: "Nur eine Geometrieart", values: [{ v: "point", d: "Punkte" }, { v: "linestring", d: "Linien" }, { v: "polygon", d: "Flächen" }] },
  { name: "dedupe", type: "boolean", default: "true", desc: "Dubletten entfernen, die durch Kachelgrenzen entstehen (gleiche Ebene und ID, sonst gleiche Eigenschaften)." },
]} />

Die Antwort ist eine `FeatureCollection`, das nächste Feature zuerst. Jedes Feature trägt seine Kachel-Attribute und zusätzlich `properties.tilequery`:

<ParamTable kind="response" params={[
  { name: "tilequery.distance", type: "number", desc: "Abstand zum Punkt in Metern (0,1 m). Polygone, die den Punkt enthalten, haben 0." },
  { name: "tilequery.geometry", type: "string", desc: "<code>point</code>, <code>linestring</code> oder <code>polygon</code>." },
  { name: "tilequery.layer", type: "string", desc: "Name der Ebene." },
  { name: "tilequery.tileset", type: "string", desc: "Nur bei mehreren Tilesets." },
]} />

Die Geometrien sind wie in der Kachel auf den Kachelrand zugeschnitten. Mit `dtm` oder `dsm` als Tileset kommt die Höhe als Punkt-Feature (`properties.elevation`).

<Code lang="bash" code={`# Gebäude und Wege im Umkreis von 30 m um den Melker Stiftsplatz
curl "https://api.mapx.at/v1/tilequery/streets/15.3338,48.2279.json?radius=30&layers=building_a,road_l"`} />

## Grundstück am Punkt

**GET https://api.mapx.at/v1/parcel/{lon},{lat}.json**

Liefert das Grundstück, das den Punkt enthält, als Polygon mit `parcel_number` (Grundstücksnummer; Bauparzellen beginnen mit einem Punkt, zum Beispiel `.14`), `cadastral_municipality_id` (Katastralgemeinde-Nummer), `area_m2` (Fläche in m²), `is_building_land` und `source`. Quelle ist die DKM des BEV (Stand der Kachelebene `cadastre`). Die früheren Felder `gstnr`, `kgnr`, `bauflaeche` und `quelle` sind deprecated und bis 31.05.2027 zusätzlich enthalten ([Migration](/api/migration/)).

- Die Katasterkacheln enthalten nur Grenzlinien. Polygon und Fläche werden daraus berechnet; die Fläche weicht um Promille vom Wert im Grundstücksverzeichnis ab.
- Grundstücke, die über das Kachelfenster um den Punkt (etwa 2 km) hinausreichen, fehlen.
- Ohne Katasterdaten am Punkt (außerhalb der gebauten Regionen, im Straßenraum ohne Grundstück) ist die Liste leer.

<Code lang="bash" code={`curl "https://api.mapx.at/v1/parcel/15.3338,48.2279.json"`} />

## Höhe am Punkt

**GET https://api.mapx.at/v1/elevation/{lon},{lat}.json**

Ein `Feature` mit `properties.dtm` (Gelände), `properties.dsm` (Oberfläche), `resolution_m` und `reference_date` (früher `stichtag`, deprecated bis 31.05.2027). Mehrere Punkte und Profile: [Höhe und Profil](/api/hoehe/).

<Code lang="bash" code={`curl "https://api.mapx.at/v1/elevation/15.3338,48.2279.json"`} />

## Fehler

`400` bei ungültiger Koordinate, Radius oder Geometrieart, `404` bei unbekanntem Tileset, `429` über dem Limit (Tilequery zählt doppelt, Grundstück vierfach). Siehe [Fehler und Limits](/api/fehler/).
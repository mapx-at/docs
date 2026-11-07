# Static Images

Kartenbilder als PNG oder WebP für Karte, Luftbild und Gelände, mit Pins, Linien und GeoJSON-Overlays.

Der Endpunkt rendert einen Kartenausschnitt als Bild, in der Form der Mapbox Static Images API. Geeignet für Vorschaubilder, E-Mails, PDF-Berichte und Seiten ohne JavaScript. Ohne Schlüssel, CORS offen.

**GET https://api.mapx.at/v1/static/{style}/{view}/{size}**

**GET https://api.mapx.at/v1/static/{style}/{overlay}/{view}/{size}**

<ParamTable params={[
  { name: "style", type: "string", required: true, desc: "Basis des Bildes", values: [
    { v: "luftbild", d: "BEV-Orthophoto, z5 bis z18" },
    { v: "gelaende", d: "Höhenstufen mit Schummerung aus dem DGM, z5 bis z16" },
  ] },
  { name: "overlay", type: "string", desc: "Kommagetrennte Liste von Overlays (siehe unten), höchstens 50 mit zusammen 20000 Stützpunkten." },
  { name: "view", type: "string", required: true, desc: "<code>lon,lat,zoom[,bearing]</code> (Drehung in Grad, Neigung nur 0) oder <code>auto</code> (nur mit Overlays, wählt den Ausschnitt passend)." },
  { name: "size", type: "string", required: true, desc: "<code>{breite}x{höhe}[@2x][.png|.webp]</code>, je 1 bis 1280 Pixel; mit <code>@2x</code> bis 2560 Pixel. Standard ist PNG." },
]} />

> Stile: `luftbild`, `gelaende` sowie die Vektorkarte `karte`, `karte-morgen`, `karte-abend`, `karte-nacht` (Flächen, Linien, Schummerung, Beschriftung; Symbole/Icons fehlen noch). `hybrid` liefert `400`.

## Overlays

| Overlay | Form | Hinweis |
|---|---|---|
| Pin | `pin-{s\|m\|l}[-label]+{hex}(lon,lat)` | Größe und Farbe; das Label wird nicht gezeichnet |
| Linie, Fläche | `path-{breite}[+{strichfarbe}[-{deckkraft}][+{füllfarbe}[-{deckkraft}]]]({polyline})` | Polyline mit Präzision 5, URL-kodiert |
| GeoJSON | `geojson({URL-kodiertes GeoJSON})` | simplestyle-Eigenschaften: `stroke`, `stroke-width`, `stroke-opacity`, `fill`, `fill-opacity`, `marker-color`, `marker-size` |

## Beispiele

<Code lang="bash" code={`# Luftbild mit rotem Pin, 600 x 400, PNG
curl -o karte.png "https://api.mapx.at/v1/static/luftbild/pin-s+ff0000(15.33,48.22)/15.33,48.22,15/600x400.png"

# Geländebild, WebP in doppelter Auflösung
curl -o gelaende.webp "https://api.mapx.at/v1/static/gelaende/12.69,47.07,12/800x600@2x.webp"

# Ausschnitt automatisch um zwei Pins
curl -o auto.png "https://api.mapx.at/v1/static/luftbild/pin-m+0000ff(15.33,48.22),pin-m+0000ff(15.35,48.24)/auto/600x400.png"`} />

In HTML genügt ein `img`-Element:

<Code lang="html" code={`<img src="https://api.mapx.at/v1/static/luftbild/15.33,48.22,15/600x400.webp" alt="Luftbild von Krummnußbaum" width="600" height="400">`} />

## Antwort und Caching

`200` mit `image/png` oder `image/webp`. Die Antwort trägt `Cache-Control` und `ETag`; mit `If-None-Match` kommt bei unverändertem Bild `304`.

## Fehler

Fehler kommen als `application/problem+json` ([RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)): `400` ungültige Eingabe (zum Beispiel Stil `hybrid`, Größe über 1280, ungültiges Overlay), `404` unbekannter Pfad, `429` Limit überschritten (`Retry-After` beachten), `503` vorübergehend nicht verfügbar. Allgemeines: [Fehler und Limits](/api/fehler/). Maschinenlesbar: [openapi.json](https://api.mapx.at/v1/openapi.json).

Für Attribution gilt [Lizenz](/lizenz/): Bilder mit Orthophoto brauchen den BEV-Hinweis.
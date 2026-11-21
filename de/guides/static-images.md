# Static-Images-Guide: Karte als Bild

Source: https://docs.mapx.at/de/guides/static-images/

> Kartenbilder als PNG oder WebP mit Pins, Pfaden und GeoJSON für Berichte, E-Mails und Webseiten erzeugen, mit der mapx-Static-Images-API.

Ein Static Image ist eine HTTP-Anfrage, die ein Bild liefert. Es eignet sich dort, wo keine interaktive Karte möglich ist: PDFs, E-Mails, Inserate, Social-Vorschauen. Der Token steht als `access_token` in der URL, weil ein Bild-Tag keinen Header senden kann. Für Webseiten einen öffentlichen Token (`mx_pk_…`) mit der eigenen Seite als erlaubter Origin verwenden; Bilder für E-Mails und PDFs auf dem eigenen Server mit einem geheimen Token (`mx_sk_…`) erzeugen, damit der geheime Token nicht in der Nachricht landet.

```html
<img alt="Wiener Innenstadt" width="600" height="400"
     src="https://api.mapx.at/v1/static/streets/16.3725,48.2085,14/600x400.png?access_token=DEIN_TOKEN">
```

## URL-Format

```
/v1/static/{style}/{view}/{width}x{height}[@2x][.png|.webp]
/v1/static/{style}/{overlay}/{view}/{width}x{height}...
```

| Teil | Werte |
|---|---|
| `style` | `streets`, `streets-dawn`, `streets-dusk`, `streets-night`, `light`, `dark`, `satellite`, `satellite-streets`, `outdoors`, `outdoors-winter` |
| `view` | `lon,lat,zoom[,bearing]` oder `auto` (passt auf die Overlays) |
| Größe | Breite und Höhe 1 bis 1280, mit `@2x` bis 2560 |
| Format | `.png` oder `.webp` |

Die Relief-Styles (`relief`, `relief-contours`, `relief-light`, `relief-dark`) gibt es nur für die Karte im Browser; als Kartenbild antwortet die API mit `404`.

Details zum Token: [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Bilder liegen in der Zone `heavy` (30 pro Minute). Sie sind cachebar: `If-None-Match` senden, dann antwortet die API mit `304`. Bilder auf der eigenen Seite zwischenspeichern und nicht pro Seitenaufruf anfordern.

## Overlays

Overlays folgen nach dem Style, getrennt durch Kommas (bis zu 50).

| Overlay | Beispiel |
|---|---|
| Pin | `pin-s+ff0000(16.3725,48.2085)` (Größe `s`, `m` oder `l`, Farbe als Hex) |
| Pfad | `path-4+1d5bd8-0.8(encodedPolyline)` (Breite, Strich, Deckkraft; Polyline-Präzision 5, URL-kodiert) |
| GeoJSON | `geojson({...})` mit simplestyle-Properties (URL-kodiert) |

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/static/streets/pin-s+ff0000(16.3725,48.2085)/16.3725,48.2085,14/600x400.png" -o vienna.png
```

## Eine Route automatisch einpassen

Mit `auto` statt der Ansicht zeigt das Bild alle Overlays.

```js
const dir = "https://api.mapx.at/v1/directions/driving/16.3725,48.2085;15.3294,48.2272?geometries=polyline&overview=simplified";
const { routes } = await (await fetch(dir, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();
const path = encodeURIComponent(routes[0].geometry); // kodierte Polyline, Präzision 5
const url = `https://api.mapx.at/v1/static/outdoors/path-5+e11d48(${path})/auto/800x500@2x.webp?access_token=DEIN_TOKEN`;
```

Die Route kommt aus [Directions](https://docs.mapx.at/de/guides/navigation.md) mit `geometries=polyline`.

## Luftbild für ein Inserat

`satellite` zeigt das Orthophoto, `outdoors` ergänzt Höhenlinien und Wege.

```
https://api.mapx.at/v1/static/satellite/pin-m+ff0000(15.3294,48.2272)/15.3294,48.2272,17/800x600.webp?access_token=DEIN_TOKEN
```

Die Quellenangabe neben dem Bild anzeigen, siehe [Attribution](https://docs.mapx.at/de/reference/attribution.md).

## Raster-Tiles eines Styles

Für Clients, die nur Raster-Tiles lesen (Leaflet, QGIS), rendern Static Tiles einen Style als 256- oder 512-px-Tiles: `/v1/styles/{id}/tiles/{256|512}/{z}/{x}/{y}`.

## Nächste Schritte

- [Static Images API](https://docs.mapx.at/de/api/static-images.md)
- [Karten](https://docs.mapx.at/de/guides/maps.md): dieselben Styles interaktiv

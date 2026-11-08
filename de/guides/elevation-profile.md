# Höhenprofil entlang einer Linie


**Ziel:** Für eine Wegstrecke ein Höhenprofil (Entfernung gegen Höhe) abfragen und als Diagramm darstellen.

**Voraussetzungen:** ein Browser. Ein kostenloser Token ([Zugang und Limits](/de/start/access-and-limits/)), kein Framework, keine Bibliothek.

Der Endpunkt [`/v1/elevation`](/de/api/elevation/) liefert bei `path` und `samples` Höhenwerte, gleichmäßig nach Länge entlang der Linie verteilt.


1. **Anfrage testen.** Die Linie besteht aus `lon,lat`-Paaren, getrennt durch Semikolon (Länge vor Breite). 50 Messpunkte entlang einer Strecke am Großglockner:

   ```bash
   curl -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/elevation?path=12.69,47.07;12.70,47.08&samples=50"
   ```

   Die Antwort ist eine GeoJSON-`FeatureCollection`, je Messpunkt ein Feature mit `elevation` (m) und `distance_m` (Strecke vom Anfang).

2. **Seite anlegen.** Eine Datei `profil.html` mit einem leeren SVG:

   ```html
   <!doctype html>
   <meta charset="utf-8">
   <svg id="profil" viewBox="0 0 600 240" style="width:100%;max-width:720px;font:12px sans-serif"></svg>
   <p id="info"></p>
   ```

3. **Profil laden und zeichnen.** Das Skript holt die Daten, skaliert Strecke und Höhe auf die Zeichenfläche und erzeugt einen Linienzug samt Fläche darunter:

   ```html
   <script type="module">
     const linie = "12.69,47.07;12.70,47.08";
     const url = `https://api.mapx.at/v1/elevation?path=${linie}&samples=100&model=dtm`;
     const daten = await (await fetch(url, { headers: { Authorization: "Bearer DEIN_TOKEN" } })).json();

     // Messpunkte ohne Höhe (außerhalb der Daten) auslassen
     const pts = daten.features
       .map(f => ({ d: f.properties.distance_m, h: f.properties.elevation }))
       .filter(p => p.h !== null);

     const B = 600, H = 240, R = { l: 50, r: 10, t: 10, b: 30 };
     const dMax = pts.at(-1).d;
     const hMin = Math.min(...pts.map(p => p.h)), hMax = Math.max(...pts.map(p => p.h));
     const x = d => R.l + (d / dMax) * (B - R.l - R.r);
     const y = h => H - R.b - ((h - hMin) / (hMax - hMin || 1)) * (H - R.t - R.b);

     const linieD = pts.map((p, i) => `${i ? "L" : "M"}${x(p.d).toFixed(1)},${y(p.h).toFixed(1)}`).join(" ");
     const flaecheD = `${linieD} L${x(dMax)},${H - R.b} L${x(0)},${H - R.b} Z`;

     document.getElementById("profil").innerHTML = `
       <path d="${flaecheD}" fill="#cfe3cf"/>
       <path d="${linieD}" fill="none" stroke="#2f6b2f" stroke-width="2"/>
       <line x1="${R.l}" y1="${H - R.b}" x2="${B - R.r}" y2="${H - R.b}" stroke="#888"/>
       <line x1="${R.l}" y1="${R.t}" x2="${R.l}" y2="${H - R.b}" stroke="#888"/>
       <text x="${R.l - 6}" y="${y(hMax) + 4}" text-anchor="end">${Math.round(hMax)} m</text>
       <text x="${R.l - 6}" y="${y(hMin) + 4}" text-anchor="end">${Math.round(hMin)} m</text>
       <text x="${R.l}" y="${H - 10}">0 km</text>
       <text x="${B - R.r}" y="${H - 10}" text-anchor="end">${(dMax / 1000).toFixed(1)} km</text>`;

     // Auf- und Abstieg aus den Messpunkten
     let auf = 0, ab = 0;
     for (let i = 1; i < pts.length; i++) {
       const dh = pts[i].h - pts[i - 1].h;
       if (dh > 0) auf += dh; else ab -= dh;
     }
     document.getElementById("info").textContent =
       `Länge ${(dMax / 1000).toFixed(2)} km, Aufstieg ${Math.round(auf)} m, Abstieg ${Math.round(ab)} m`;
   </script>
   ```

4. **Öffnen.** `profil.html` über einen statischen Server öffnen (`npx serve`); der Browser darf die API direkt abfragen. Für Webseiten dazu einen öffentlichen Token (`mx_pk_…`) mit der eigenen Origin verwenden, `localhost` nur während der Entwicklung erlauben.


## Ergebnis

Ein SVG-Diagramm mit Fläche und Linie der Höhe über der Strecke und darunter Länge, Aufstieg und Abstieg.

> Je mehr `samples`, desto genauer der Auf- und Abstieg, aber desto mehr Rauschen. Mit `model=dsm` zeigt das Profil die Oberfläche (Bäume, Gebäude) statt des Geländes, siehe [DTM und DSM](/de/start/faq/#was-ist-der-unterschied-zwischen-dtm-und-dsm). Ein Pfad hat 2 bis 512 Stützpunkte, `samples` 2 bis 1024.


## Nächste Schritte

- Alle Parameter und Antwortfelder: [Höhe und Profil](/de/api/elevation/).
- Linie aus einem GPX-Track: Koordinaten als `lon,lat;lon,lat` zusammensetzen, Reihenfolge siehe [Koordinaten](/de/reference/coordinates/).
- Limits bei vielen Abfragen: [Fehler und Limits](/de/api/errors/).


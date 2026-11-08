# Terrain-RGB

Wie mapx Höhen als Farbe kodiert, Wertebereich, Rundung und Dekodierfunktionen.

Terrain-RGB speichert eine Höhe pro Pixel in den drei Farbkanälen eines Bildes. So lassen sich Höhenkacheln mit gewöhnlichen
Bildformaten ausliefern, und Clients wie MapLibre berechnen daraus Schummerung und 3D-Gelände.

## Formel

```
höhe_m = -10000 + (R · 65536 + G · 256 + B) · 0,1
```

R ist das höchstwertige Byte. Der Zahlenwert `q = R·65536 + G·256 + B` liegt zwischen 0 und 16 777 215, die Höhe also zwischen
**−10 000 m und +1 667 721,5 m** in Schritten von 0,1 m.

## Dekodieren

<Code lang="js" code={`const hoehe = (r, g, b) => -10000 + (r * 65536 + g * 256 + b) * 0.1;`} />
<Code lang="python" code={`hoehe = -10000 + (r.astype("uint32") * 65536 + g * 256 + b) * 0.1  # r, g, b als numpy-Arrays`} />

## Worauf achten

- **Bildformat verlustfrei lesen.** mapx liefert WebP lossless. Jede verlustbehaftete Zwischenstufe (JPEG, Farbprofil, Premultiplied Alpha, Skalierung beim Dekodieren) verfälscht die Höhe.
- **Nicht interpolieren, bevor dekodiert ist.** Mischen von RGB-Werten ergibt keine gemischte Höhe. Erst dekodieren, dann mitteln.
- **Mapbox-kompatibel.** `encoding: "mapbox"` in MapLibre. Terrarium (`R·256 + G + B/256 − 32768`) ist eine andere Kodierung und wird von mapx nicht ausgeliefert.
- **512 Pixel, kein Rand.** `tileSize: 512`.

Anwendung: [DTM Geländemodell](/de/tilesets/terrain-rgb/).
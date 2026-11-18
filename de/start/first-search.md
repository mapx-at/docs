# Erste Suche: Geocoding-Anfrage

Source: https://docs.mapx.at/de/start/first-search/

> Eine Adresse oder einen Ort in Österreich mit einer Geocoding-Anfrage finden (curl, JavaScript, Python) und das Ergebnis als Marker auf einer Karte zeigen.

## Anfrage

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1" | jq '.features[0] | {place_name, center}'
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
const { features } = await res.json();
console.log(features[0].place_name, features[0].center); // center ist [lon, lat]
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode", params={"q": "Stephansplatz 1, Wien", "types": "address", "limit": 1}, headers={"Authorization": "Bearer DEIN_TOKEN"})
feature = res.json()["features"][0]
print(feature["place_name"], feature["center"])
```

Die Antwort ist eine GeoJSON-`FeatureCollection`. Jedes Feature hat `place_name`, `center` (`lon,lat`) und `geometry`. `types=address` beschränkt die Ergebnisse auf Adressen; ohne den Parameter kann bei einer Hausnummer auch eine Straße oder ein Ort vorne liegen.

## Ergebnis auf der Karte zeigen

Diesen Code auf der Seite aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) nach `new maplibregl.Map(...)` einfügen (die Karte in `const map` speichern):

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
const [hit] = (await res.json()).features;
new maplibregl.Marker().setLngLat(hit.center).addTo(map);
map.flyTo({ center: hit.center, zoom: 16 });
```

Den Code in ein `<script type="module">` setzen, damit Top-Level-`await` funktioniert.

## Nächste Schritte

- [Such-Guide](https://docs.mapx.at/de/guides/search.md): Type-ahead, Kategorien, Adressformen
- [Geocoding API](https://docs.mapx.at/de/api/geocoding.md)

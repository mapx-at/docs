# Erste Suche: Geocoding-Anfrage

Source: https://docs.mapx.at/de/start/first-search/

> Eine Adresse oder einen Ort in Österreich mit einer Geocoding-Anfrage finden (curl, JavaScript, Python) und das Ergebnis als Marker auf einer Karte zeigen.

## Anfrage

**curl**

```bash
curl -s -H "Authorization: Bearer DEIN_TOKEN" "https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1" | jq '.features[0] | {name: .properties.name, place_formatted: .properties.place_formatted, coordinates: .geometry.coordinates}'
```

**JavaScript**

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
const { features } = await res.json();
console.log(features[0].properties.name, features[0].geometry.coordinates); // coordinates ist [lon, lat]
```

**Python**

```python
import requests

res = requests.get("https://api.mapx.at/v1/geocode", params={"q": "Stephansplatz 1, Wien", "types": "address", "limit": 1}, headers={"Authorization": "Bearer DEIN_TOKEN"})
feature = res.json()["features"][0]
print(feature["properties"]["name"], feature["properties"]["place_formatted"], feature["geometry"]["coordinates"])
```

Die Antwort ist eine GeoJSON-`FeatureCollection`; das `curl`-Beispiel liefert zum Beispiel `{"name":"Stephansplatz 1","place_formatted":"Innere Stadt, 1010 Wien, W","coordinates":[16.372605,48.208728]}`. Bei `401` oder `403` siehe [Zugang und Limits](https://docs.mapx.at/de/start/access-and-limits.md). Jedes Feature hat eine `id`, eine Punkt-`geometry` (`[lon, lat]`) und in `properties` die Felder `type`, `name`, `place_formatted`, `relevance`, `accuracy` und `address` (Straße, Hausnummer, PLZ, Ort … einzeln); Search Box und Autofill antworten im selben Format. `types=address` beschränkt die Ergebnisse auf Adressen; ohne den Parameter kann bei einer Hausnummer auch eine Straße oder ein Ort vorne liegen.

## Ergebnis auf der Karte zeigen

Diesen Code auf der Seite aus [Erste Karte](https://docs.mapx.at/de/start/first-map.md) nach `new maplibregl.Map(...)` einfügen (die Karte in `const map` speichern). Er steht im `<script type="module">`, nur dort funktioniert Top-Level-`await`:

```js
const res = await fetch("https://api.mapx.at/v1/geocode?q=Stephansplatz%201,%20Wien&types=address&limit=1", { headers: { Authorization: "Bearer DEIN_TOKEN" } });
const [hit] = (await res.json()).features;
new maplibregl.Marker().setLngLat(hit.geometry.coordinates).addTo(map);
map.flyTo({ center: hit.geometry.coordinates, zoom: 16 });
```

## Nächste Schritte

- [Such-Guide](https://docs.mapx.at/de/guides/search.md): Type-ahead, Kategorien, Adressformen
- [Geocoding API](https://docs.mapx.at/de/api/geocoding.md)
